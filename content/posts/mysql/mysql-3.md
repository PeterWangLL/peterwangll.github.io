---
title: Limit&Offset——分页慢查询优化
date: 2024-10-17
categories:
  - Technology
tags:
  - MySQL
nolastmod: true
cover: /img/pic-12.jpg
draft: true
---

```sql
select* from article LIMIT 1,3
select * from article LIMIT 3 OFFSET 1
```
上面两种写法都表示取第2,3,4三条数据。

limit的偏移量越大，执行时间越长。
### 原因分析
eg. user表有id,name,sex,age字段，其中id为主键，age为普通索引。当执行`select * from user where age > 10 limit 1000000,10`时：
1. 首先SQL的执行顺序是：from->join->where->group by->having->select->distinct->order by->limit.
2. 由于age是普通索引，而select的是*，所以需要回表。
3. limit是在最后执行的，所以需要进行1000010次回表，然后把前1000000条数据抛弃，取后10条数据。
由此可见造成上述情况的罪魁祸首是`回表`。
### 优化
1. 可以通过子查询或覆盖索引来减少回表，根据业务来决定。
2. order by和limit结合使用。
3. 主键直接定位(ID需有序)，分步查询/程序记录上一页最大主键ID。