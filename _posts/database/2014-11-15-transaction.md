---
layout: post
title: 事务隔离级
categories: db
tags : transaction acid
---

### 事务隔离级  

数据库中的基本锁机制：排它锁（写锁，X锁）和共享锁（读锁，S锁）  

**排它锁**（写锁，X锁）事务T对数据对象A加了X锁，其他事务就不能对A加锁了，直到T释放A上的锁，这样就保证了其他事务在T释放A上的锁之前不能读或者写A    
**共享锁**（读锁，S锁）事务T对数据对象A加了S锁，可以对读A但是不能写A，其他事务可以对A加S锁，读A，但是不能加X锁，写A，直到事务T释放A上的锁。  


#### 事务的ACID特性    

1.  A 原子性　atomic 事务要么全部完成，要么全部取消，即使它持续运行10个小时。如果事务崩溃，状态回到事务之前（事务回滚）
2.  C 一致性  consistency 只有合法的数据（依照关系约束和函数约束）能写入数据库，一致性与原子性和隔离性有关
3.  I 隔离性  isolation 事务的执行不受其他并发事务的的影响。
4.  D 持久性  durability 一旦事务提交（也就是成功执行）,不管发生什么（崩溃或者出错），数据要保存在数据库中。

#### mysql innodb的事务隔离级

MySQL InnoDB存储引擎，实现的是基于多版本的并发控制协议——MVCC (Multi-Version Concurrency Control) (注：与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control)。
MVCC最大的好处，相信也是耳熟能详：读不加锁，读写不冲突。在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能，这也是为什么现阶段，几乎所有的RDBMS，都支持了MVCC。??

在MVCC并发控制中，读操作可以分成两类：快照读 (snapshot read)与当前读 (current read)。快照读，读取的是记录的可见版本 (有可能是历史版本)，不用加锁。
当前读，读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。??

快照读：简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)

    select * from table where ?;

当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。

    select * from table where ? lock in share mode;
    select * from table where ? for update;
    insert into table values (…);
    update table set ? where ?;
    delete from table where ?;

MySQL/InnoDB定义的4种隔离级别：

1.  Read Uncommited可以读取未提交记录。此隔离级别，不会使用，忽略。
2.  Read Committed (RC)

    2.1 快照读忽略，本文不考虑。

    2.2针对当前读，RC隔离级别保证对读取到的记录加锁 (记录锁)，存在幻读现象。
3.  Repeatable Read (RR)

    3.1快照读忽略，本文不考虑。

    3.2针对当前读，RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)，不存在幻读现象。
4.  Serializable    从MVCC并发控制退化为基于锁的并发控制。不区别快照读与当前读，所有的读操作均为当前读，读加读锁 (S锁)，写加写锁 (X锁)。Serializable隔离级别下，读写冲突，因此并发度急剧下降，在MySQL/InnoDB下不建议使用。

事务隔离级 在没有设置GLOBAL和SESSION时,隔离级的设置只对当前事务有效  

所有的*写*操作都是在内存中，只有当commit时才写回磁盘。

结束事务的两种不同的方式：回滚或者提交

### 参考

[SET TRANSACTION Syntax]<http://dev.mysql.com/doc/refman/5.7/en/set-transaction.html#isolevel_read-committed>

[Consistent NonLocking Reads]<http://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html>

[locking read]<http://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read>

[mysql 加锁处理分析]<http://hedengcheng.com/?p=771>
