---
title: Overview of MySQL
date: 2024-09-07
categories:
  - Technology
tags:
  - MySQL
nolastmod: true
cover: /img/pic-10.jpg
draft: true
---
从一条SQL语句的执行出发
1. Server：连接-分析-优化-执行，重点是优化器
2. 执行器与InnDB的交互，索引下推
3. B+Tree
4. Buffer Pool
5. log
6. MVCC
7. 锁机制

从架构的角度看，Server+Engine，Server层看优化器的原理，执行器承担的是调用存储引擎接口，过滤组装的工作。\
由于InnoDB索引存储的特点，MySQL5.6支持索引下推，当命中联合索引时，将筛选过程下推至存储引擎，从而减轻执行器的压力。\
InnoDB对数据的操作都是在Buffer Pool中的，以页为单位与物理磁盘交互，构建出逻辑表空间，当在内存中对数据进行操作前，先记录undo log，完成后记录redo log，bin log，这涉及两阶段提交，当事务提交后，不管修改的页是否刷盘，理论上都能保证能够查到数据，但还要看redo log刷盘策略。\
MVCC的实现基于undo log和行记录的隐藏字段的，通过构建Read View来描述事务可见性。\
在更新或者当前读时，要求数据是最新的，因而需要锁机制：加锁的粒度、S｜X锁、MDL，最终是对索引加锁。

这是我目前对MySQL全貌的认识，执行器、InnoDB层面都大量地使用了内存，对于Buffer Pool，redo log等日志分别对应了不同的刷盘策略；从SQL执行的过程来看优化，其实就是通过高效的索引模型，以最精简的结构返回数据，不要粗暴地把事情全部交给MySQL去做。