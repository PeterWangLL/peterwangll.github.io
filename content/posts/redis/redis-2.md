---
title: Redis发布订阅
date: 2024-10-18
categories:
  - Technology
tags:
  - Redis
nolastmod: true
cover: /img/pic-18.jpg
draft: true
---

### 简介
Redis通过`SUBSCRIBE`和`PUBLISH`实现发布订阅。主要由三部分组成：「Subscriber」、「Publisher」、「Channel」。通过Pattern模式可以实现订阅多个前缀相同的频道。
### 原理
Redis使用pubsub_channels字典来保存订阅频道的信息(value是一个链表)
### 缺点
1. 发布的消息并不会持久化，消息发布之后新订阅的客户端只能接收后续发布到该频道的消息。
2. 不支持ACK机制，无法保证消息的可靠性。