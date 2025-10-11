---
title: Why is Redis so fast?
date: 2024-10-09
categories:
  - Technology
tags:
  - Redis
nolastmod: true
cover: /img/pic-20.jpg
draft: true
---
本文会从以手稿+文字的形式说明，仅为个人总结。
本文会讲述上半部分：包括数据结构、单线程模型、IO多路复用三个方面阐明Redis为什么那么快？

首先Redis基于内存存储。这是一个前提，下面会谈到正是基于内存，Redis才会采用某某设计，而不是另外的那些。

## 数据结构
SDS，Dict（Hash），ZipList，SkipList，QuickList，ListPack。

SDS详细的可以看<http://zhangtielei.com/posts/blog-redis-sds.html>\
通过header维护了一个len，alloc，flags，buf[]的头信息，len取代了`‘\0’`发挥的作用，因此SDS是二进制安全的，获取len复杂度是O(1)的，再因为会在拼接时检查(dsMakeRoomFor)，所以它没有缓冲区溢出的风险。

Hash使用拉链法，看过JDK HashMap实现就问题不大。扩容机制不太一样，Dict维护两个Hash表，当触发rehash阈值后，只有对某个桶进行增删改查操作后，才会将该桶对应的key-value迁移到另外一个Hash表中，通过这种渐进式的做法防止一次性rehash带来的阻塞问题。


ZipList压缩链表是一种对空间的极致利用，在头里面会存节点熟练，尾部偏移等字段，entry节点有prevlen，encoding和data，prevlen用于记录前一个节点的长度，方便从后向前遍历，成也萧何败也萧何，当插入节点后导致prevlen变化，会发生连锁更新。同样由于自身存储结构限制，压缩链表不能存储太多的元素，在Redis ZSet实现中，当元素个数小于128个并且每个元素的值小于64字节时会使用ZipList。

QuickList通过双向链表模式，维护的节点变成了一个个压缩链表，通过控制每个节点压缩链表的大小来规避潜在的连锁更新风险。

ListPack重新设计了压缩链表，将prevlen字段去除彻底解决了连锁更新问题，而从后向前遍历的功能没有丢失，详细的算法可以看<https://github.com/antirez/listpack/blob/master/listpack.c>中的IpDecodeBacklen函数。

SkipList跳表，层级结构和skipListNode节点如上，通过随机数保持相邻两层节点个数比2:1。

会有人拿它和B+Tree比较，InnoDB就是基于B+Tree实现的，它的前提的磁盘，意味要最大限度减少磁盘IO，由此B+Tree矮胖的特点良好的适应了这种需求，而SkipList在相同数据量下明显比B+Tree层数多，Why？上面提到了Redis基于内存这一前提，层数多一些又何妨，况且跳表比B+Tree少了维护树的成本，实现起来也简单，何乐不为呢。

## IO多路复用
从计算机视角看，IO实际上是数据在内存和外部设备（磁盘、网络）之间的流动。\
如果是一次Input，数据会从外部存储流入内核缓冲区（下面统称为A过程），再从内核缓冲区流入用户进程缓冲区（下面统称为B过程），Output相反。\
IO多路复用是对传统NIO模型的改进，BIO对于从A过程是阻塞的状态，而NIO通过轮询的方式查看A过程是否完成，于是IO多路复用更进一步，将这一轮询查看的工作统一交给一个线程去做，从宏观的角度看复用了这条IO。\
于是不得不提到select/epoll，简单来说select就是一个数组轮询，poll用链表替代了数组，本质仍是轮询，epoll通过事件驱动机制进一步提升了效率。Reactor模式就是这种思想。

## 单线程
通过优秀的数据结构，IO多路复用，内存，Redis的高性能瓶颈并非CPU，可能是机器的内存或网络带宽，因此单线程足矣，美中不足无法利用CPU多核优势，可以在一台机器上启动多个节点或采用分片集群的方式。\
Redis6.0引入多线程，目的是处理网络IO，因为随着网络硬件性能的提升，Redis的性能瓶颈有时会出现在网络IO上。