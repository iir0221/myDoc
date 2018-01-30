# Mysql
```sh
brew services start mysql (启动)
brew services stop mysql (停止)
```

# 锁
## 表锁
ALTER TABLE
## 行级锁

# 事务
mysql事务由存储引擎实现，不要在事务中混合使用事务型和非事务型的表(InnoDB和MyISAM)
## 事务SQL
```sql
start transaction;
select ename from employee where eid=2;
update employee set ename = "aaa" where eid=2;
commit;
```

atomicity

consisitency

isolation

durability

## 隔离级别--SQL标准中定义的

READ UNCOMMITTED(未提交读)--脏读

READ COMMITED(提交读)--不可重复读--多数数据库默认隔离级别
```
不可重复读：在同一事务中，两次读取同一数据，得到内容不同
    事务1：查询一条记录
                    -------------->事务2：更新事务1查询的记录
                    -------------->事务2：调用commit进行提交
    事务1：再次查询上次的记录
    
    ***此时事务1对同一数据查询了两次，可得到的内容不同，称为不可重复读
```

REPEATABLE READ(可重复读)--mysql默认事务隔离级别--幻读
```
幻读：同一事务中，用同样的操作读取两次，得到的记录数不相同
    事务1：查询表中所有记录
                        -------------->事务2：插入一条记录
                        -------------->事务2：调用commit进行提交
    事务1：再次查询表中所有记录
    
    ***此时事务1两次查询到的记录是不一样的，称为幻读
    详细解释：
        幻读是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，
        这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表
        中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，
        就好象发生了幻觉一样。
```
SERIALIZABLE(可串行化)--导致大量超时


## 死锁
InnoDB处理死锁的方法是：将持有最少行级排他锁的事务进行回滚

## 事务日志

Write-Ahead Logging

## mysql中的事务

支持事务的存储引擎
* InnoDB
* NDB Cluster

### 自动提交AUTOCOMMIT--默认
每个语句被当做一个事务来处理

### 设置隔离级别
```sql
SET TRANSACTION ISOLATION LEVEL
```
新的隔离级别在下一次事务开始时生效


可以设置整个数据库的隔离级别，也可以只改变当前会话(Session)的隔离级别
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

会话Session与连接Connection
> 通俗来讲，会话(Session) 是通信双方从开始通信到通信结束期间的一个上下文（Context）。这个上下文是一段位于服务器端的内存：记录了本次连接的客户端机器、通过哪个应用程序、哪个用户登录等信息。

> 连接（Connection）：连接是从客户端到Oracle实例的一条物理路径。连接可以在网络上建立，或者在本机通过IPC机制建立。通常会在客户端进程与一个专用服务器或一个调度器之间建立连接。

> 会话(Session) 是和连接(Connection)是同时建立的，两者是对同一件事情不同层次的描述。简单讲，连接(Connection)是物理上的客户端同服务器的通信链路，会话(Session)是逻辑上的用户同服务器的通信交互。


## 隐式和显式锁定


上述描述的锁都是隐式锁，InnoDB会根据隔离级别在需要的时候自动加锁

InnoDB也支出通过特定的语句显示锁定，这些语句不属于SQL规范

```sql
SELECT ... LOCK IN SHARE MODE
SELECT ... FOR UPDATE
```

mysql 在服务器层面实现支持LOCK TABLES和UNLOCAK TABLES语句，和存储引擎无关，但是不推荐使用它们。

# 多版本并发控制MVCC
大多数数据库都实现了MVCC

MVCC可以保证非阻塞的读操作，写操作只锁定必要的行。

其原理大致是保存数据在某个时间点的**快照**，这样对于一个事务来说，不管它执行了多久，它看到的数据是一样的。

## InnoDB的实现



