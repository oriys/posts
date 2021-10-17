---
title: "八股文::数据库"
date: 2021-08-22T21:51:33+08:00
draft: false
---

## 数据库

### MySQL 为什么使用 B+ 树来作索引，对比 B 树它的优点和缺点是什么？

由于 mysql 通常将数据存放在磁盘中，读取数据就会产生磁盘 IO 消耗。而 B+ 树的非叶子节点中不保存数据，B 树中非叶子节点会保存数据，通常一个节点大小会设置为磁盘页大小，这样 B+树每个节点可放更多的 key，B 树则更少。这样就造成了，B 树的高度会比 B+ 树更高，从而会产生更多的磁盘 IO 消耗。

B+ 树叶子节点构成链表，更利用范围查找和排序。而 B 树进行范围查找和排序则要对树进行递归遍历

B+ 树层级更少，查找更快
B+ 树查询速度稳定：由于 B+ 树所有数据都存储在叶子节点，所以查询任意数据的次数都是树的高度 h
B+ 树有利于范围查找
B+ 树全节点遍历更快：所有叶子节点构成链表，全节点扫描，只需遍历这个链表即可
B 树优点：如果在 B 树中查找的数据离根节点近，由于 B 树节点中保存有数据，那么这时查询速度比 B+树快。

### 数据库的事务隔离级别有哪些？各有哪些优缺点？

- 读未提交
  - 读写都不加锁，不隔离
  - 每次查询到的都是最新版
  - 引发脏读问题
- 读提交
  - 读此时候已经提交的数据
  - 写数据时，加 X 锁，提交时释放
  - Oracle 默认隔离级别
- 可重复读
  - 读取事务开始的数据状态
  - 写数据时，加 X 锁，提交时释放
  - MySQL 默认隔离级别
- 串行化
  - 读数据加 S 锁，写数据时加 X 锁，提交时释放
  - 对于同一条数据，同时只有一个事务进行写操作
  - 隔离性最高，性能最差

### 简述乐观锁以及悲观锁的区别以及使用场景

- 悲观锁
  - 每次获取数据的时候，都会担心数据被修改，所以每次获取数据的时候都会进行加锁，确保在自己使用的过程中数据不会被别人修改，使用完成后进行数据解锁。由于数据进行加锁，期间对该数据进行读写的其他线程都会进行等待。
- 乐观锁
  - 每次获取数据的时候，都不会担心数据被修改，所以每次获取数据的时候都不会进行加锁，但是在更新数据的时候需要判断该数据是否被别人修改过。如果数据被其他线程修改，则不进行数据更新，如果数据没有被其他线程修改，则进行数据更新。由于数据没有进行加锁，期间该数据可以被其他线程进行读写操作。
- 使用场景
  - 悲观锁
    - 比较适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样会增加大量的锁的开销，降低了系统的吞吐量
  - 乐观锁
    - 比较适合读取操作比较频繁的场景，如果出现大量的写入操作，数据发现冲突的可能性会加大，为了保证数据的一致性，应用层需要不断的重现获取数据，这样会增加大量的查询操作，降低了系统的吞吐量。

### 产生死锁的必要条件有哪些？如何解决死锁？

- 四个条件
  - 互斥条件：一个资源每次只能被一个进程使用。
  - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
  - 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
  - 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。
- 解决
  - 按同一顺序访问对象:如果所有并发事务按同一顺序访问对象，则发生死锁的可能性会降低。
  - 避免事务中的用户交互:避免编写包含用户交互的事务，因为运行没有用户交互的批处理的速度要远远快于用户手动响应查询的速度
  - 保持事务简短并在一个批处理中:在同一数据库中并发执行多个需要长时间运行的事务时通常发生死锁。事务运行时间越长，其持有排它锁或更新锁的时间也就越长，从而堵塞了其它活动并可能导致死锁。保持事务在一个批处理中，可以最小化事务的网络通信往返量，减少完成事务可能的延迟并释放锁。
  - 使用低隔离级别:确定事务是否能在更低的隔离级别上运行。执行提交读允许事务读取另一个事务已读取（未修改）的数据，而不必等待第一个事务完成。使用较低的隔离级别（例如提交读）而不使用较高的隔离级别（例如可串行读）可以缩短持有共享锁的时间，从而降低了锁定争夺。
  - 使用绑定连接:使用绑定连接使同一应用程序所打开的两个或多个连接可以相互合作。次级连接所获得的任何锁可以象由主连接获得的锁那样持有，反之亦然，因此不会相互阻塞。

### 聚簇索引和非聚簇索引有什么区别？

聚簇索引是对磁盘上实际数据重新组织以按指定的一个或多个列的值排序的算法。特点是存储数据的顺序和索引顺序一致。一般情况下主键会默认创建聚簇索引，且一张表只允许存在一个聚簇索引。聚簇索引的叶子节点就是数据节点，而非聚簇索引的叶子节点仍然是索引节点，只不过有指向对应数据块的指针。

InnoDB 的的二级索引的叶子节点存放的是 KEY 字段加主键值。因此，通过二级索引查询首先查到是主键值，然后 InnoDB 再根据查到的主键值通过主键索引找到相应的数据块。而 MyISAM 的二级索引叶子节点存放的还是列值与行号的组合，叶子节点中保存的是数据的物理地址。所以可以看出 MYISAM 的主键索引和二级索引没有任何区别，主键索引仅仅只是一个叫做 PRIMARY 的唯一、非空的索引，且 MYISAM 引擎中可以不设主键。

### 什么是数据库事务，MySQL 为什么会使用 InnoDB 作为默认选项

数据库事务是指一个逻辑单元执行的一系列操作，一个逻辑工作单元必须有四个属性，称为 ACID（原子性、一致性、隔离性和持久性）属性

- 以主流的 MyISAM 和 InnoDB 的对比
  - 功能对比
    - InnoDB 支持 ACID 的事务 4 个特性，而 MyISAM 不支持；
    - InnoDB 支持 4 种事务隔离级别，默认是可重复读 repeatable read，MyISAM 不支持；
    - InnoDB 支持 crash 安全恢复，MyISAM 不支持；InnoDB 支持外键，MyISAM 不支持；
    - InnoDB 支持行级别的锁粒度，MyISAM 不支持，只支持表级别的锁粒度；
    - InnoDB 支持 MVCC，MyISAM 不支持。
    - InnoDB 特性上，InnoDB 表最大可以 64TB，支持聚簇索引、支持压缩数据存储，支持数据加密，支持查询/索引/数据高速缓存，支持自适应 hash 索引、空间索引，支持热备份和恢复等。
- 性能对比
  - 读写混合模式下，随着 CPU 核数的增加，InnoDB 的读写能力呈线性增长，在这个测试用例里，最高可达近 9000 的 TPS，但 MyISAM 因为读写不能并发，它的处理能力跟核数没关系，呈一条水平线，TPS 低于 500。
  - 只读模式下，随着 CPU 核数的增加，InnoDB 的读写能力呈线性增长，最高可达近 14000 的 TPS，但 MyISAM 的处理能力不到 3000。

### 简述什么是最左匹配原则

最左优先，以最左边的为起点任何连续的索引都能匹配上

skip scan 之后所谓的最左原则还成立吗？

### MySQL 中 InnoDB 和 MylSAM 的区别是什么？

[MySQL 存储引擎 InnoDB 与 Myisam 的六大区别 | 菜鸟教程](https://www.runoob.com/w3cnote/mysql-different-nnodb-myisam.html)

### 数据库如何设计索引，如何优化查询？

[MySQL 如何设计索引更高效？ - SegmentFault 思否](https://segmentfault.com/a/1190000038921156)

### 简述数据库中什么情况下进行分库，什么情况下进行分表？

[这四种情况下，才是考虑分库分表的时候！ - SegmentFault 思否](https://segmentfault.com/a/1190000038944473)

### 什么是 SQL 注入攻击？如何防止这类攻击？

[SQL 注入攻击原因及预防 - SegmentFault 思否](https://segmentfault.com/a/1190000023211211)

### MySQL 有哪些常见的存储引擎？它们的区别是什么？

[MySQL :: MySQL 8.0 Reference Manual :: 16 Alternative Storage Engines](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)

### 简述一致性哈希算法的实现方式及原理

[大话「一致性哈希」](https://morven.life/posts/consistent_hash/)

### MySQL 中 join 与 left join 的区别是什么？

[MySQL :: MySQL 8.0 Reference Manual :: 13.2.10.2 JOIN Clause](https://dev.mysql.com/doc/refman/8.0/en/join.html)

### 联合索引的存储结构是什么？

[联合索引在 B+树上的存储结构及数据查找方式 - 掘金](https://juejin.cn/post/6844904073955639304)

### 简述 MySQL 常见索引类型，介绍一下覆盖索引

[MySQL :: MySQL 8.0 Reference Manual :: 13.1.15 CREATE INDEX Statement](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)

### 数据库索引的实现原理是什么？

[MySQL 索引原理及慢查询优化 - 美团技术团队](https://tech.meituan.com/2014/06/30/mysql-index.html)

### 简述 MySQL MVCC 的实现原理

- 行记录的版本控制
  - 由于 redo log 的存在，可以从最新版本推算出之前的版本
- 快照度
  - 不锁定数据的情况下，读取数据的特定历史版本
  - 版本由事物的具体需求确定
    - RC：根据每次 SELECT 时，其他事务的提交情况
    - RR：根据事物开始时，其他事物的提交情况
- 当前读
  - 读取数据的最新版本，并加锁
  - 若当前版本被加锁且不兼容，则阻塞等待
  - X 锁: UPDATE，DELETE，SELECT FOR UPDATE
  - S 锁: SELECT IN SHARE MODE

### 数据库的读写分离的作用是什么？如何实现？

[MySQL 读写分离 — Danna 的博客](https://dannashen.github.io/2019/07/04/MySQL%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB/)

### 如何解决缓存与数据库不一致的问题？

[Consistency between Redis Cache and SQL Database | Yunpeng's Blog](https://yunpengn.github.io/blog/2019/05/04/consistent-redis-sql/)

### MySQL 的索引什么情况下会失效？

[MySQL :: MySQL 8.0 Reference Manual :: 8.3.1 How MySQL Uses Indexes](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)

### MySQL 联合索引底层原理是什么？

[MYSQL 索引的底层原理【图文】](https://blog.51cto.com/u_15317888/3232479)

### 唯一索引与普通索引的区别是什么？使用索引会有哪些优缺点？

[唯一索引与普通索引的区别是什么？使用索引会有哪些优缺点？ | Jamin 的个人博客](http://blog.xuanweiyao.com/archives/3b3c800b6b8648f2aae88f61dac30b53)

### 简述事务的四大特性

- 原子性(Atomicity)
  - 事务要么全部成功，要么全部失败
  - MySQL 的两段式提交保证了事务的原子性
  - undo log 用来回滚事物的更改
- 一致性(Consistency)
  - 数据库要从一个一致性状态变换到另一个一致性状态
  - 锁和两段式提交保证了一致性
- 隔离性(Isolation)
  - 事务不能被其他事务的操作数据干扰
  - 多个并发事务之间要相互隔离
  - 锁和 undo log 实现了 MySQL 事务的隔离性
- 持久性(Durability)
  - 一个事务一旦被提交，它对数据库的数据的改变就是永久的
  - redo log 实现了 MySQL 事务的持久化

### 如何解决幻读的问题

- 间隙锁的功能与行锁相同，只是针对间隙加锁
- 间隙锁不分读写，不允许在间隙内插入
- 可重复读加锁时候，同时锁住数据和所有间隙

### 简述间隙锁的作用

- 加锁时以 Next-Key(间隙加上下一行记录)为基本单位
- 查找过程中扫描过的范围才加锁
- 唯一索引等值查询，没有间隙锁，只加行锁
- 索引等值查询最右一个扫描到的不满足条件值不加行锁
- 索引才盖上且只加 S 锁时，不锁主键索引

### 简述更新流程

- 执行器先找引擎取要更新的记录，如果这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，给这行加上 X 锁，然后再返回。
- 执行器拿到引擎给的行数据，把这个值更新，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
- 引擎先将这个更新操作记录到 redo log 里面，再将这行新数据更新到内存中的数据页，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
- 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
- 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，解 X 锁，更新完成。

### B+ 树中叶子节点存储的是什么数据

聚簇索引是数据，二级索引是主键索引的引用

### MySQL 索引使用什么数据结构？

B+ 树

### SQL 优化的方案有哪些，如何定位问题并解决问题？

[常见性能优化策略的总结](https://tech.meituan.com/2016/12/02/performance-tunning.html)

### 简述主从复制以及读写分离的使用场景

### 假设建立联合索引 (a, b, c) 如果对字段 a 和 c 查询，会用到这个联合索引吗？

会

### MySQL 有什么调优的方式？

[MySQL 性能调优的 10 个方法 - 知乎](https://zhuanlan.zhihu.com/p/39038788)

### MySQL 常用的聚合函数有哪些？

[MySQL :: MySQL 8.0 Reference Manual :: 12.20.1 Aggregate Function Descriptions](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html)

### 简述数据库事务复制原理

[MySQL 复制原理\_焦玉丽-CSDN 博客](https://blog.csdn.net/JYL15732624861/article/details/82493502)

### 什么时候索引会失效？

### 如何解决主从不一致的问题？

- 问题
  - log 传送开销较小，主要是消费 relay log 耗时
  - 备库性能不如主库
  - 备库承担了很多分析 SQL
  - 主库的长事务未提交
- 处理方法
  - 主备使用相同配置的机器
  - 备库关闭 log 实时沙盘
  - 增加从库数量，应对分析 SQL
  - binlog 传送至大数据系统，供分析
  - 大事务一分多
  - 多线程分配执行 relay log
    - 按表分发
    - 按行分发
    - 按事务组分发

### 数据库主键索引和唯一索引有什么区别？

### 简述 binlog、undo log 和 redo log 的作用

- Binlog 归档日志
  - Binlog 是 server 层产生的逻辑日志
  - 用来进行数据复制和数据传送
  - Binlog 完整记录了数据库每次的数据操作，可作为数据闪回手段
  - Binlog 记录在专门的文件中
- Undo Log 回滚日志
  - InnoDB 自身产生的逻辑日志，用于事务回滚和展示旧版本
  - 对任何数据 (缓存) 的更新，都先写 Undo Log
  - Undo Log 位于表空间的 undo segment 中
  - SQL:`update name = 'b'` -> Undo Log:`update name = 'a'`
- Redo Log 重做日志
  - InnoDB 自身产生的物理日志，记录数据页的变化
  - InnoDB “日志优先于数据”，记录 Redo Log 视为数据已经更新
  - 内存中的数据更新后写 Redo Log，数据被写入硬盘后删除
  - Redo Log 储存在 4 个 1GB 文件中，并且循环写入
  - write pos 当前日志写入点
  - check point 是擦出点，数据被更新到硬盘时擦除
  - 当 write pos 追上 check pos 点时，事务无法提交，需要等待 check point 推进
  - Redo Log 保证了数据的持久化

### 模糊查询是如何实现的？

### MySQL 中 varchar 和 char 的区别是什么？

[MySQL :: MySQL 8.0 Reference Manual :: 11.3.2 The CHAR and VARCHAR Types](https://dev.mysql.com/doc/refman/8.0/en/char.html)

### 如何设计数据库压测方案？

### 数据库查询中左外连接和内连接的区别是什么？

左外连接的结果集包含左表的所有记录和右表中满足连接条件的记录，结果集中那些不符合连接条件的来源于右表的列值为 null

内连接查询会将 T1 表的每一行和 T2 表的每一行进行比较,并找出满足连接谓词的组合。当连接谓词被满足，A 和 B 中匹配的行会按列组合(并排组合)成结果集中的一行。

### 并发事务会引发哪些问题？如何解决？

### 数据库索引的叶子结点为什么是有序链表？

为了范围查找

### 如何定位以及优化数据库慢查询

[数据库优化（一）Mysql 慢查询的定位和分析 - 知乎](https://zhuanlan.zhihu.com/p/164897114)

### 建立了三个单列索引 a, b, c 查询 where a = ? and b = ? and c = ？索引会起作用吗？

可能会，如果走全表的代价小于走索引，那就走全表。

### 数据库反范式设计会出现什么问题？

[数据中设计中的范式与反范式 · Mysql 设计与优化专题 · 看云](https://www.kancloud.cn/thinkphp/mysql-design-optimalize/39324)

### MySQL 锁的种类

按照粒度分，MySQL 锁可以分为全局锁、表级锁、行锁

- 全局锁
  - FTWRL(Flush tables with read lock)
  - 使整个库处于只读状态
  - 主要用途保证备份的一致性
- 表锁
  - lock tables table_name read/write
  - 锁住整张表
  - 元数据锁 (matadata lock)
    - 表的结构、字段，数据类型、索引
    - 事务访问元数据时，会自动给表加 DML 读锁
    - 事物修改元数据时，会自动给表加 MDL 写锁
- 行锁
  - 读锁/写锁
  - 只有写写兼容

全局锁会锁住左右表，整个库无法修改

表级锁分为表锁 (数据锁) 和元数据锁

行锁会锁住数据行，分为共享锁和独占锁

### 什么是脏读、不可重复读、幻读

- 脏读: 读到了其他事务未提交的数据
- 不可重复读: 同样的查询读到的数据内容不一样
- 幻读: 同样的查询读到了更多的数据

MySQL 不同隔离级别的问题

| 隔离级别 | 脏读 | 不可重复读 |  幻读  |
| -------- | :--: | :--------: | :----: |
| RU       |  X   |     X      |   X    |
| RC       |  √   |     X      |   X    |
| RR       |  √   |     √      | 部分 √ |
| SR       |  √   |     √      |   √    |

### 长事务危害

- 行级锁长时间无法释放，导致其他事务等待
  - 当前读会对数据行加锁，事务提交前无法释放
  - 调整 innodb_lock_wait_timeout 参数，默认 50 秒，超过 50 秒还未获取锁报错，适当缩短参数
- 容易产生死锁
  - 两个事务都等待对方的锁释放
  - 主动死锁检测: innodb_deadlock_detect，发现死锁回滚代价比较小的事务
- MDL 锁 hold 住大量事务，造成 MySQL 奔溃
  - 事务访问数据时，会自动给表加 MDL 锁
  - 事务修改元数据时，自动加 MDL 锁
  - 遇到锁不兼容时，申请 MDL 锁的事务形成一个队列
- 查看影响性能的锁
  - 查看长事务 infomation_schema 库 innodb_trx 表
  - 查看锁 infomation_schema 库 innodb_locks 表
  - 查看阻塞的事务 infomation_schema 库 innodb_locks_waits 表
  - 查看锁 performance_schema 库 data_locks 表
  - 查看锁等待 performance_schema 库 data_locks_waits 表
  - 查看 MDL 锁 performance_schema 库 matadata_locko 表
- 业务上的建议
  - 控制长事务，没有必要的情况不开始事务
  - 数据修改尽量放在事务后部分，降低锁时间

### 数据库备份

- 备份时数据库的状态
  - 热备：正常运行中直接备份
  - 冷备：完全停止后备份
  - 温备：数据库只读
- 备份文件的格式
  - 逻辑备份：导出文本或 SQL 语句
  - 物理备份：备份数据库底层文件
- 备份文件的内容
  - 完全备份：备份完整数据
  - 增量备份：备份数据差异
  - 日志备份：备份 Binlog
- 工具
  - mysqldump：逻辑，热备，全量或部分
    - 特点
      - 非常常用的 MySQL 逻辑备份工具
      - MySQL Server 自带
      - 输出的备份内容为 SQL 语句，平衡了阅读和还原
      - SQL 语句占空间较小
    - 原理
      - `SELECT SQL_NO_CACHE * FROM table_name`，避免干扰其他事务，不会进入 SQL 缓存
    - 使用方法
      - 导出 `mysqldump -uroot -p123456 --database db --single-transaction > db.sql`
      - 导入 `source db.sql`
    - 缺点
      - 导出逻辑数据，备份慢
      - 导入需要执行 SQL 语句，备份慢
    - 增量备份
      - 思路：binglog 忠实记录了 MySQL 数据的变化，mysqldump 全量备份之后，可以用 binlog 作为增量，mysqldump 全量备份时，切换到新的 binlog 文件
      - `mysqldump -uroot -p123456 --database db --single-transaction --flush-logs --master-data=2> db.sql`
  - xtrabackup：物理，热，全量 + 增量备份
    - 特点
      - 直接备份 InnoDB 底层数据文件
      - 导出不需要转换，速度快
      - 工作时对数据库压力小
      - 容易实现增量备份
    - 实现
      - 启动 redo log 监听线程，开始收集 redo leg
      - 拷贝 ibd 数据文件
      - 停止收集 redo log
      - 加 FTWRL 锁拷贝元数据 frm，一小段温备时间
    - 还原
      - 还原 ibd，重放 redo log
    - 使用
      - 备份 `innobackupex --user==root --password=123456 backdir/`
      - 还原 `innobackupex --copy-back bakdir/xxxx-xx-xx/`
      - 增量备份 `innobackupex --user=root --password=123456 --incremental bakdir/ --increamental-basedir='bakdir/xxxx-xx-xx/'`
      - 合并增量到全量 `innobackupex --apply-log bakdir/xxxx-xx-xx/ --incremental-dir=bakdir/yyyy-yy-yy/`
  - 其他
    - mylvmbackup
      - 物理、温备、利用 LVM、直接备份磁盘数据
    - mydumper
      - 对比 mysqldump 实现了多线程并发
    - Zmanda Recovery Manage
      - 集成了多种功能备份工具
      - 集成了 binlog 分析工具
      - 图形化界面
