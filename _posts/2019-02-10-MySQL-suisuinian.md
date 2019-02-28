---
layout: post
title: MySQL 碎碎念
categories:
- MySQL
feature_image: "https://picsum.photos/2560/600?image=350"
---


### mysql MDL读锁

- MDL锁（metadata lock），用来保护表的元数据信息，主要用于解决或者保证DDL操作与DML操作之间的一致性。  
- sessionA的CRUD会加上MDL读锁，而sessionB的DDL加MDL写锁，要等前面所有的锁释放完才能轮到DDL操作，后面所有此表的操作都会被阻塞。
- [参考文章](http://www.ywnds.com/?p=7209&viewuser=42)

### RR隔离级别下快照时间
- RR隔离级别下，事务执行过程中当遇到第一个普通读后，整个事务就开始使用当前最新的快照，这个快照是针对所有的表，而不是你读操作的表

| 序号        | 事务A   |  事务B  |
| --------   | -----:  | ----:  |
| 1      | select * from A   |        |
| 2        |      |   update B set b=5 where id = 6  |
| 3        |    select * from B where id = 6    |    |
| 4       |    select * from B where id = 6    |    |

- 事务A中第3步读到的数据仍然是B更新前的数据
- 事务A中第4步由于是当前读所以可以读到最新数据


### 当前读与幻读
- MySQL当前读(如for update等)在RR隔离级别下也会读到其它事务提交的数据，所以可能产生幻读。
- 普通读由于读取的都是快照数据，所以不会产生幻读的情况
- [参考文章](https://www.cnblogs.com/renolei/p/5910060.html)

### 主键锁
- MySQL主键锁并不是事务锁，申请完就释放
- 对于 insert  into select 这种类型的，要在事务结束后才释放，因为不知道有多少条，申请方式是按次申请，每次条数为  2的(n-1)方，所以可能产生id不连续。
