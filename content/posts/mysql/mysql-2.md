---
title: Multi-Versioned Concurrency Control
date: 2024-09-09 17:52:19
categories:
  - Technology
tags:
  - MySQL
nolastmod: true
cover: /img/pic-11.jpg
draft: true
---
多版本并发控制，在事务开启时，构建Read View——逻辑上的一致性视图。

通过行记录上的隐藏字段db_roll_ptr(指向undo)和db_trx_id(对该行记录最后操作的事务ID)，以及在创建事务的那一时刻，未提交但活跃的事务数来实现MVCC。

事务创建时只需要新增对应的未提交但活跃的事务数组，最底位、最高位(事务id时递增的)，来实现秒级创建。db_roll_ptr维护了一个undo的链式结构，通过它可以找到对应时刻的行记录。