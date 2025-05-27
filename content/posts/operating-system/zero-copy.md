---
title: 零拷贝
date: 2024-10-19
categories:
  - Technology
tags:
  - OS
nolastmod: true
cover: /img/pic-19.jpg
---

从用户进程缓冲区到内核缓冲区，需要1次系统调用（2次上下文切换），反之亦然。最初外部设备到内核缓冲区也需要CPU亲力亲为，但是磁盘速度太慢了，对CPU是一种极大的浪费，于是有了DMA（Direct Memory Access）直接内存访问技术，将CPU解放出来。
### 没有DMA之前
没有DMA技术之前，IO过程是这样的：
* CPU发出对应的指令给磁盘控制器
* 磁盘控制器把数据放入到内部缓冲区
* CPU收到中断信号后将数据从磁盘控制器缓冲区拷贝到PageCache
* 再将数据从PageCache拷贝到用户进程缓冲区
### DMA
直接内存访问(Direct Memory Access)：在进行IO设备和内存的数据传输的时候，数据搬运的工作全部交给DMA控制器，即DMA负责将数据从磁盘控制器缓冲区拷贝到内核缓冲区。
### 传统的文件传输
数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的IO接口从磁盘读取或写入。一般会需要两个系统调用：
`read(file, tmp_buf, len)`和`write(socket, tmp_buf, len)`
这个过程发生了4次用户态与内核态的上下文切换，还发生了4次数据拷贝。
### 零拷贝
##### mmap+write
用`mmap()`替换`read()`系统调用函数，`mmap()`会直接把内核缓冲区里的数据「映射」到用户空间，这样就减少了一次数据拷贝。
##### sendfile
Linux2.1提供了一个专门发送文件的系统调用函数`sendfile()`，可以直接把内核缓冲区里的数据拷贝到socket缓冲区里，这样只要一次系统调用。
##### SG-DMA
如果网卡支持SG-DMA(The Scatter-Gather Direct Memory Access)技术，缓冲区描述符和数据长度传到socket缓冲区，这样网卡的SG-DMA控制器就可以直接将内核缓冲区的数据拷贝到网卡到缓冲区里。