# Undo log
```       
          rollback-->atomicity
undo log
          MVCC-->repeatable read(isoloation)
          
          事务结束后事务所占undo——随事务一起过期
```
# Redo log:重建赃页
* 每一次事务都将产生一个LSN
* 提交一个事务的流程
  ```      
  trx start-->write redo log buffer-->write undo log-->write db buffer
  -->commit-->write log file-->trx end
  -->checkpoint-->write db disk
  ```  
  * redo log 从buffer刷新到file的时机
    * Master Thread每一秒将redo log buffer刷新到redo log file
    * 每个事务提交时，将redo log buffer刷新到redo log file
    * 当redo log缓冲池剩余空间小于1/2时（默认redo log缓冲池大小为8M），将redo log buffer刷新到redo log file
  * 赃页刷新到磁盘的时机[（checkpoint)](./checkpoint.md)
  * 事务提交时
* redo log的作用（优点）
  * 磁盘随机跟新效率低下
    * 如果每次修改数据和索引都需要更新到磁盘，必定会大大增加I/O请求，因为每次更新的位置都是随机的，磁头需要频繁定位导致效率低；redo log提升了读的效率。
    * redo log的另一个作用是，通过延迟dirty page的flush最小化磁盘的random writes。（redo log会合并一段时间内TRX对某个page的修改）
    * log file是连续的，所以在提交的时候写log file，在checkpoint时，写db disk（离散的）效率较高
  * redo log durability
    * 在每次事务commit的时候，就立刻将事务更改操作记录到redo log（之后这次事务才算完成）。所以即使buffer pool中的dirty page在断电时丢失，InnoDB在启动时，仍然会根据redo log中的记录完成数据恢复。
          
          




# 【msql】关于redo 和 undo log【转载】
转载自[http://www.cnblogs.com/chenpingzhao/p/5003881.html]
* undo log：保证事务的原子性以及InnoDB的MVCC
* redo log：保证事务的持久性。
* Write Ahead Log： 和大多数关系型数据库一样，InnoDB记录了对数据文件的物理更改，并保证总是日志先行，也就是所谓的WAL（Write Ahead Log），即在持久化数据文件前，保证之前的redo日志已经写到磁盘

## 一、概念
### 1、Innodb Crash Recovery
这是InnoDB引擎的一个特点，当故障发生，重新启服务后，会自动完成恢复操作，将数据库恢复到之前一个正常状态（不需要重做所有的日志，只需要执行上次刷入点之后的日志，这个点就叫做Checkpoint）恢复过程有两步

第一步：检查redo日志，将之前完成并提交的事务全部重做；

第二步：将undo日志中，未完成提交的事务，全部取消

### 2、LSN
LSN(log sequence number) 用于记录日志序号，它是一个不断递增的 unsigned long long 类型整数。

在 InnoDB 的日志系统中，LSN 无处不在，它既用于表示修改脏页时的日志序号，也用于记录checkpoint，通过LSN，可以具体的定位到其在redo log文件中的位置。

LSN 用字节偏移量来表示。每个page有LSN，redo log也有LSN，Checkpoint也有LSN。可以通过命令show engine innodb status来观察：
```
---
LOG
---
Log sequence number 602146258
Log flushed up to   602146258
Pages flushed up to 602146258
Last checkpoint at  602146249
0 pending log flushes, 0 pending chkp writes
1050 log i/o's done, 0.00 log i/o's/second
```
为了管理脏页，在 Buffer Pool 的每个instance上都维持了一个flush list，flush list 上的 page 按照修改这些 page 的LSN号进行排序。因此定期做redo checkpoint点时，选择的 LSN 总是所有 bp instance 的 flush list 上最老的那个page（拥有最小的LSN）。**由于采用WAL的策略，每次事务提交时需要持久化 redo log 才能保证事务不丢。而延迟刷脏页则起到了合并多次修改的效果，避免频繁写数据文件造成的性能问题。**

### 3、Dirty page
在InnoDB中，写到bp里面的数据一方面可以加快数据处理速度，同时也会造成数据的不一致(RAM vs DISK)

1、在对用户每次有导致数据变更的请求中，Innodb引擎把数据和索引都载入到内存中的缓冲池(buffer pool)中，如果每次修改数据和索引都需要更新到磁盘，必定会大大增加I/O请求，因为每次更新的位置都是随机的，磁头需要频繁定位导致效率低；数据暂放在内存中，也一定程度的提高了读的速度。所以Innodb每处理完一个请求(Transaction)后只添加一条日志log，另外有一个线程负责智能地读取日志文件并批量更新到磁盘上，实现最高效的磁盘写入。innodb既然利用Mem buffer提高相应的速度，那当然也会带来数据不一致，术语为脏数据，mysql称之为dirty page。

**发生过程：当事务(Transaction)需要修改某条记录（row）时，InnoDB需要将该数据所在的page从disk读到buffer pool中，事务提交后，InnoDB修改page中的记录(row)。这时buffer pool中的page就已经和disk中的不一样了，mem中的数据称为脏数据（dirty page）。**

2、**在每次事务commit的时候，就立刻将事务更改操作记录到redo log。所以即使buffer pool中的dirty page在断电时丢失，InnoDB在启动时，仍然会根据redo log中的记录完成数据恢复，redo log的另一个作用是，通过延迟dirty page的flush最小化磁盘的random writes。（redo log会合并一段时间内TRX对某个page的修改）**

redo log buffer-->redo log file--> db buffer--> db disk

3、正常情况下，dirty page什么时候flush到disk上

* redo log是一个环(ring)结构，当redo空间占满时，将会将部分dirty page flush到disk上，然后释放部分redo log。这种情况可以通过Innodb_log_wait(SHOW GLOBAL STATUS)观察，情况发生该计数器会自增一次
* 当需要在Buffer pool分配一个page，但是已经满了，并且所有的page都是dirty的（否则可以释放不dirty的page），通常是不会发生的。这时候必须flush dirty pages to disk。这种情况将会记录到Innodb_buffer_pool_wait_free中。一般地，可以通过启动参数innodb_max_dirty_pages_pct控制这种情况，当buffer pool中的dirty page到达这个比例的时候，将会强制设定一个checkpoint，并把dirty page flush到disk中
* 检测到系统空闲的时候，会flush，每次64 pages

> 涉及的InnoDB配置参数：innodb_flush_log_at_trx_commit、innodb_max_dirty_pages_pct；状态参数：Innodb_log_wait、Innodb_buffer_pool_wait_free 

4、dirty page既然是在Buffer pool中，那么如果系统突然断电Dirty page中的数据修改是否会丢失

例如如果一个用户完成一个操作（数据库完成了一个事务，page已经在buffer pool中修改，但dirty page尚未flush），这时系统断电，buffer pool数据全部消失。那么这个用户完成的操作（导致的数据库修改）是否会丢失呢？答案是不会(innodb_flush_log_at_trx_commit=1)。这就是redo log要做的事情，在disk上记录更新（buffer pool中的数据并不是永久性）

系统故障造成数据库不一致的原因有两个：

* 未完成事务对数据库的更新可能已写入数据库
* 已提交事务对数据库的更新可能还留在缓冲区没来得及写入数据库

在这里我们先说恢复的一般方法：

* 正向扫描日志文件（从头到尾），找出故障发生前已经提交的事务（存在begin transaction和commit记录），将其标识记入重做（redo）队列。同时找出故障发生时未完成的事务（只有begin transaction，没commit），将其标识记入（undo）队列
* 对undo队列的各事务进行撤销处理。进行undo的处理方法是，反向扫描日志文件，对每个undo事务的更新操作执行反操作，即将日志记录中“更新前的值”写入数据库
* 对重做日志中的各事务进行重做操作。进行redo的处理方法是，正向扫描日志，对每个redo事务重新执行日志文件登记操作。即将日志中“更新后的值”写入数据库
#### innodb_flush_log_at_trx_commit  

默认值1的意思是每一次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时。特别是使用电池供电缓存（Battery backed up cache）时。设成2对于很多运用，特别是从MyISAM表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。日志仍然会每秒flush到硬盘，所以一般不会丢失超过1-2秒的更新。设成0会更快一点，但安全方面比较差，即使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统挂了时才可能丢数据。 

#### innodb_max_dirty_pages_pct

his is an integer in the range from 0 to 100. The default value is 90. The main thread in InnoDB tries to write pages from the buffer pool so that the percentage of dirty (not yet written) pages will not exceed this value. 

### 4、ACID
* 原子性(Atomicity)：事务中的所有操作，要么全部完成，要么不做任何操作，不能只做部分操作。如果在执行的过程中发生了错误，要回滚(Rollback)到事务开始前的状态，就像这个事务从来没有执行过

* 事务的持久性(Durability)：事务一旦完成，该事务对数据库所做的所有修改都会持久的保存到数据库中。为了保证持久性，数据库系统会将修改后的数据完全的记录到持久的存储上

### 5、Log buffer
日志在内存里也是有缓存的，这里将其叫做log buffer。磁盘上的日志文件称为log file。log file一般是追加内容，可以认为是顺序写，顺序写的磁盘IO开销要小于随机写

当buffer cache中的数据块被修改后，服务器进程生成redo数据并写入到redo log buffer中。当满足以下条件时，LGWR会将redo log buffer中的条目开始写入在线重做日志：

* redo log buffer满1/3
* 每3秒超时（Timeout
* log_buffer中的数据到达1M
* 事务提交时

如果我们的系统拥有快速处理器和I/O相对较慢的磁盘,处理器可能会填满缓存区的其余空间，这时会促使LGWR移动缓冲区的部分数据到磁盘。在这种情况下，较大的日志缓冲区能够临时掩盖 较慢磁盘对系统带来的影响。你这可做下面的选择：

* 提升checkpoint或归档进程
* 提升LGWR性能（也许你可以将所有的在线重做日志放置到速度更快的裸设备）

#### 合理使用redo log buffer
执行批量操作时使用批量提交，以至LGWR能够更高效的写入重做条目到在线重做日志文件中
当加载大量数据时，使用nologging操作

#### 设置Log Buffer
redo log buffer由初始化参数LOG_BUFFER决定，修改该参数需要重启实例。适当的redo log buffer参数值能够明显的提升系统吞吐量，尤其对于插入，更新，删除大数据量的系统。默认的log buffer值：MAX(0.5M,(128K*number of cpus))，通常默认值是足够的。增加log buffer的值对系统性能或可恢复性不会产生负面影响，而仅仅会使用额外的内存。

log_buffer一般在3-5M就足够了。超过3-5M，仅仅是浪费内存；当然太小了，也可能影响性能。在内存不太昂贵的今天，且如果你有大量“大事务”，log_buffer就设定为5M吧。

## 二、Undo log
Undo log是InnoDB MVCC事务特性的重要组成部分。当我们对记录做了变更操作时就会产生undo记录，Undo记录默认被记录到系统表空间(ibdata)中，但从5.6开始，也可以使用独立的Undo 表空间

**Undo记录中存储的是老版本数据，当一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。当版本链很长时，通常可以认为这是个比较耗时的操作（例如bug#69812）。**

大多数对数据的变更操作包括INSERT/DELETE/UPDATE，其中INSERT操作在事务提交前只对当前事务可见，因此产生的Undo日志可以在事务提交后直接删除（谁会对刚插入的数据有可见性需求呢！！），而对于UPDATE/DELETE则需要维护多版本信息，在InnoDB里，UPDATE和DELETE操作产生的Undo日志被归成一类，即update_undo。

### 原理
Undo Log的原理很简单，为了满足事务的原子性，在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为Undo Log）。然后进行数据的修改。如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。

除了可以保证事务的原子性，Undo Log也可以用来辅助完成事务的持久化。

Undo Log 是为了实现事务的原子性，在MySQL数据库InnoDB存储引擎中，还用Undo Log来实现多版本并发控制(简称：MVCC)。

用Undo Log实现原子性和持久化的事务的简化过程
假设有A、B两个数据，值分别为1,2。
```
A.事务开始.
B.记录A=1到undo log.
C.修改A=3.
D.记录B=2到undo log.
E.修改B=4.
F.将undo log写到磁盘。
G.将数据写到磁盘。
H.事务提交
```
这里有一个隐含的前提条件：‘数据都是先读到内存中，然后修改内存中的数据，最后将数据写回磁盘’,之所以能同时保证原子性和持久化，是因为以下特点：
```
A. 更新数据前记录Undo log。
B. 为了保证持久性，必须将数据在事务提交前写到磁盘。只要事务成功提交，数据必然已经持久化。
C. Undo log必须先于数据持久化到磁盘。如果在G,H之间系统崩溃，undo log是完整的，可以用来回滚事务。
D. 如果在A-F之间系统崩溃,因为数据没有持久化到磁盘。所以磁盘上的数据还是保持在事务开始前的状态。
```
缺陷：每个事务提交前将数据和Undo Log写入磁盘，这样会导致大量的磁盘IO，因此性能很低，如果能够将数据缓存一段时间，就能减少IO提高性能。但是这样就会丧失事务的持久性。

## 三、Redo Log
Innodb的事务日志是指Redo log，保存在日志文件ib_logfile*里面

redo log可以通过参数innodb_log_files_in_group配置成多个文件，另外一个参数innodb_log_file_size表示每个文件的大小。因此总的redo log大小为innodb_log_files_in_group * innodb_log_file_size，Redo log文件以ib_logfile[number]命名，日志目录可以通过参数innodb_log_group_home_dir控制。

Redo log 以顺序的方式写入文件文件，写满时则回溯到第一个文件，进行覆盖写。（但在做redo checkpoint时，也会更新第一个日志文件的头部checkpoint标记，所以严格来讲也不算顺序写）

Redo log文件是循环写入的，在覆盖写之前，总是要保证对应的脏页已经刷到了磁盘。在非常大的负载下，Redo log可能产生的速度非常快，导致频繁的刷脏操作，进而导致性能下降，通常在未做checkpoint的日志超过文件总大小的76%之后，InnoDB 认为这可能是个不安全的点，会强制的preflush脏页，导致大量用户线程stall住。如果可预期会有这样的场景，我们建议调大redo log文件的大小。可以做一次干净的shutdown，然后修改Redo log配置，重启实例。

原理
和Undo Log相反，Redo Log记录的是新数据的备份。在事务提交前，只要将Redo Log持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是Redo Log已经持久化。系统可以根据Redo Log的内容，将所有数据恢复到最新的状态。

Undo + Redo事务的简化过程 
假设有A、B两个数据，值分别为1,2.
```
A.事务开始.
B.记录A=1到undo log.
C.修改A=3.
D.记录A=3到redo log.
E.记录B=2到undo log.
F.修改B=4.
G.记录B=4到redo log.
H.将redo log写入磁盘。
I.事务提交
```
Undo + Redo事务的特点
```
  A. 为了保证持久性，必须在事务提交前将Redo Log持久化。
  B. 数据不需要在事务提交前写入磁盘，而是缓存在内存中。
  C. Redo Log 保证事务的持久性。
  D. Undo Log 保证事务的原子性。
  E. 有一个隐含的特点，数据必须要晚于redo log写入持久存储。
```
## 四、IO性能
Undo + Redo的设计主要考虑的是提升IO性能。虽说通过缓存数据，减少了写数据的IO,但是却引入了新的IO，即写Redo Log的IO。如果Redo Log的IO性能不好，就不能起到提高性能的目的。为了保证Redo Log能够有比较好的IO性能，InnoDB 的 Redo Log的设计有以下几个特点：

* A. 尽量保持Redo Log存储在一段连续的空间上。因此在系统第一次启动时就会将日志文件的空间完全分配。以顺序追加的方式记录Redo Log,通过顺序IO来改善性能。
* B. 批量写入日志。日志并不是直接写入文件，而是先写入redo log buffer.当需要将日志刷新到磁盘时(如事务提交),将许多日志一起写入磁盘.
* C. 并发的事务共享Redo Log的存储空间，它们的Redo Log按语句的执行顺序，依次交替的记录在一起，以减少日志占用的空间。例如,Redo Log中的记录内容可能是这样的：
```
记录1: <trx1, insert …>
记录2: <trx2, update …>
记录3: <trx1, delete …>
记录4: <trx3, update …>
记录5: <trx2, insert …>
```
* D. 因为C的原因,当一个事务将Redo Log写入磁盘时，也会将其他未提交的事务的日志写入磁盘。
* E. Redo Log上只进行顺序追加的操作，当一个事务需要回滚时，它的Redo Log记录也不会从Redo Log中删除掉。

## 五、恢复(Recovery)
在recovery里面可以看到log是非常必要的：当数据库发生异常的时候，数据是可以恢复的。对于不是损坏磁盘驱动器的异常，恢复是自动进行的。InnoDB读取最新的checkpoint日志记录，检查dirty pages是否在异常发生前写到磁盘上了，如果没有，则读取影响该页的log记录并应用它们。这被称为”rolling forward”。因为有LSN，所以InnoDB只需要比较这个数字就可以进行同步。
### 恢复策略
前面说到未提交的事务和回滚了的事务也会记录Redo Log，因此在进行恢复时,这些事务要进行特殊的的处理.有2中不同的恢复策略：

A. 进行恢复时，只重做已经提交了的事务。
B. 进行恢复时，重做所有事务包括未提交的事务和回滚了的事务。然后通过Undo Log回滚那些未提交的事务。

当你使用UPDATE, INSERT, DELETE语句更新数据的时候，你就改变了两个地方的数据：log buffer和data buffers。Buffers是固定长度的内存块，通常是512字节。
```
LOG BUFFER           DATA BUFFER
=================    ===============
= Log Record #1 =    = Page Header =
= Log Record #2 =    = Data Row    =
= Log Record #3 =    = Data Row    =
= Log Record #4 =    = Data Row    =
=================    ===============
```
例如：INSERT INTO JOBS VALUES(1,2,3)语句执行之后，log buffer将增加一个新的log记录，称为Log Record #5，它包含一个rowid和新记录的内容。同时，data buffer也将增加一个新行，但是，它会同时在页头标识：该页最新的log记录是Log Record #5。在这个例子中#5是Log Sequence Number（LSN），它对于接下来操作的时序安排是至关重要的。
 
下面是data-change的一些细节：
 
* 1.一个INSERT log记录仅包含一个新数据，它对于在页上重做操作是足够的了，因此被称为一个redo条目。
* 2.LSN不是log记录的一个域，它是文件中的一个绝对地址的相对偏移值。
 
在InnoDB改变了log buffer和data buffer之后，接下来就是写盘了。这就是复杂的地方。有多个线程在监控buffer的活动情况，有三种情况――overflow， checkpoint和commit――可以导致写盘操作。
### Overflows情况下发生了什么
Overflow是很少发生的情况，因为InnoDB采用pro-active措施来防止buffers被填满。但是我们还是来看看下面两种情况：

* 1.如果log buffer满了，InnoDB在buffer的末尾写log。那么情况向下面的图一样（log buffer只有四条记录的空间，现在插入第五条记录）：
    ```
    LOG FILE(S) BEFORE WRITING LOG RECORD #5
    =================
    = Log Record #1 =
    = Log Record #2 =
    = Log Record #3 =
    = Log Record #4 =
    =================

    LOG FILE(S) AFTER WRITING LOG RECORD #5
    =================
    = Log Record #5 =
    = Log Record #2 =
    = Log Record #3 =
    = Log Record #4 =
    =================
    ```
    logs不可能永远增长。即使InnoDB使用了某些压缩算法，log文件还是会由于太大而不能放到任何磁盘驱动器上。因此InnoDB采取循环写的办法，也就是说将会覆盖前面就的log记录。
* 2.如果data buffer满了，InnoDB将最近使用的buffer写入到数据库中，但是不可能足够的快。这种情况下，页头的LSN就起作用了，InnoDB检查它的LSN是否比log文件中最近的log记录的LSN大，只有当log赶上了data的时候，才会将数据写到磁盘。换句话说，数据页不会写盘，直到相应的log记录需要写盘的时候。这就是先写日志策略。

### CheckPoint的时候发生了什么
Checkpoint机制每次刷新多少页，从哪里取脏页，什么时间触发刷新？这些都是很复杂的。有两种Checkpoint，分别为：Sharp Checkpoint、Fuzzy Checkpoint

Sharp Checkpoint发生在关闭数据库时，将所有脏页刷回磁盘

Fuzzy Checkpoint在运行时使用进行部分脏页的刷新，部分脏页刷新有以下几种：

#### A、Master Thread Checkpoint
Master Thread以每秒或每十秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘。这个过程是异步的，不会阻塞查询线程。

#### B、FLUSH_LRU_LIST Checkpoint
InnoDB要保证LRU列表中有100左右空闲页可使用。在InnoDB1.1.X版本前，要检查LRU中是否有足够的页用于用户查询操作线程，如果没有，会将LRU列表尾端的页淘汰，如果被淘汰的页中有脏页，会强制执行Checkpoint刷回脏页数据到磁盘，显然这会阻塞用户查询线程。从InnoDB1.2.X版本开始，这个检查放到单独的Page Cleaner Thread中进行，并且用户可以通过innodb_lru_scan_depth控制LRU列表中可用页的数量，默认值为1024。

#### C、Async/Sync Flush Checkpoint
是指重做日志文件不可用时，需要强制将脏页列表中的一些页刷新回磁盘。这可以保证重做日志文件可循环使用。在InnoDB1.2.X版本之前，Async Flush Checkpoint会阻塞发现问题的用户查询线程，Sync Flush Checkpoint会阻塞所有查询线程。InnoDB1.2.X之后放到单独的Page Cleaner Thread。

#### D、Dirty Page too much Checkpoint
脏页数量太多时，InnoDB引擎会强制进行Checkpoint。目的还是为了保证缓冲池中有足够可用的空闲页。其可以通过参数innodb_max_dirty_pages_pct来设置：

```sql
mysql> show variables like 'innodb_max_dirty_pages_pct';
```
```
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_max_dirty_pages_pct | 90    |
+----------------------------+-------+
```
前面说过InnoDB采取了一些pro-active措施来保证不发生overflows，其中最重要的措施就是checkpointing。有一个分离的线程，或者说从一组修改buffers的线程中分离出来的一个线程。在特定的时间间隔，checkpointer将醒来，检查buffer的改变，并保证写盘操作已经发生了。

大部分DBMS在这个时候，将会把所有的buffer写盘，这样可以保证所有改变了但是没写盘的buffer都写盘。就是说DBMS将通过”Sharp Checkpoint” flush所有”dirty”buffers。但是InnoDB只保证：

* a、log和data buffers不会超过某个限制点；

* b、log始终比data先写盘；

* c、没有哪个data buffer的页头LSN等于被覆盖写的log记录。也就是说InnoDB是”Fuzzy Checkpoint”。

**在COMMIT的时候，InnoDB不会将dirty data page写盘。之所以强调这个是因为，很容易让人想到，提交改变就是将所有东西写到一个持久媒介上。其实，只有log记录需要写。写dirty data page只可能发生在overflow或checkpoint时刻，因为它们的内容是多余的。**
 
 
参考文章

http://www.orczhou.com/index.php/2009/08/innodb-dirty-page-redo-log-2/
http://mysqldump.azundris.com/archives/78-Configuring-InnoDB-An-InnoDB-tutorial.html

http://dev.mysql.com/doc/refman/5.0/en/innodb.html

http://mysql.taobao.org/monthly/2015/06/01/

http://mysql.taobao.org/monthly/2015/04/01/

http://mysql.taobao.org/monthly/2015/05/01/

http://my.oschina.net/jockchou/blog/478162

http://www.mysqlops.com/2011/12/20/understanding_index2.html













# 理解数据库中的undo日志、redo日志、检查点【转载】

数据库存放数据的文件，本文称其为data file。
数据库的内容在内存里是有缓存的，这里命名为db buffer。某次操作，我们取了数据库某表格中的数据，这个数据会在内存中缓存一些时间。对这个数据的修改在开始时候也只是修改在内存中的内容。当db buffer已满或者遇到其他的情况，这些数据会写入data file。

## undo，redo
日志在内存里也是有缓存的，这里将其叫做log buffer。磁盘上的日志文件称为log file。log file一般是追加内容，可以认为是顺序写，顺序写的磁盘IO开销要小于随机写。

**Undo日志记录某数据被修改前的值，可以用来在事务失败时进行rollback；Redo日志记录某数据块被修改后的值，可以用来恢复未写入data file的已成功事务更新的数据。** 下面的示例来自于杨传辉《大数据分布式存储系统 原理解析与架构实践》，略作改动。

> 例如某一事务的事务序号为T1，其对数据X进行修改，设X的原值是5，修改后的值为15，那么Undo日志为<T1, X, 5>，Redo日志为<T1, X, 15>。

也有把undo和redo结合起来的做法，叫做Undo/Redo日志，在这个例子中Undo/Redo日志为<T1, X, 5, 15>。

当用户生成一个数据库事务时，undo log buffer会记录被修改的数据的原始值，redo会记录被修改的数据的更新后的值。

**redo日志应首先持久化在磁盘上，然后事务的操作结果才写入db buffer，（此时，内存中的数据和data file对应的数据不同，我们认为内存中的数据是脏数据），db buffer再选择合适的时机将数据持久化到data file中。这种顺序可以保证在需要故障恢复时恢复最后的修改操作。先持久化日志的策略叫做Write Ahead Log，即预写日志。**

redo log buffer-->redo log file--> db buffer

在很多系统中，undo日志并非存到日志文件中，而是存放在数据库内部的一个特殊段中。本文中就把这些存储行为都泛化为undo日志存储到undo log file中。

对于某事务T，在log file的记录中必须开始于事务开始标记（比如“start T”），结束于事务结束标记（比如“end T”、”commit T”）。在系统恢复时，如果在log file中某个事务没有事务结束标记，那么需要对这个事务进行undo操作，如果有事务结束标记，则redo。

在db buffer中的内容写入磁盘数据库文件之前，应当把log buffer的内容写入磁盘日志文件。

有一个问题，redo log buffer和undo log buffer存储的事务数量是多少，是按照什么规则将日志写入log file？如果存储的事务数量都是1个，也就意味着是将日志立即刷入磁盘，那么数据的一致性很好保证。在执行事T时，突然断电，如果未对磁盘上的redo log file发生追加操作，可以把这个事务T看做未成功。如果redo log file被修改，则认为事务是成功了，重启数据库使用redo log恢复数据到db buffer和 data file即可。

如果存储多个的话，其实也挺好解释的。就是db buffer写入data file之前，先把日志写入log file。这种方式可以减少磁盘IO，增加吞吐量。不过，这种方式适用于一致性要求不高的场合。因为如果出现断电等系统故障，log buffer、db buffer中的完成的事务会丢失。以转账为例，如果用户的转账事务在这种情况下丢失了，这意味着在系统恢复后用户需要重新转账。

## 检查点checkpoint
checkpoint是为了定期将db buffer的内容刷新到data file。当遇到内存不足、db buffer已满等情况时，需要将db buffer中的内容/部分内容（特别是脏数据）转储到data file中。**在转储时，会记录checkpoint发生的”时刻“。在故障回复时候，只需要redo/undo最近的一次checkpoint之后的操作。**

## 幂等性问题
在日志文件中的操作记录应该具有幂等性。幂等性，就是说同一个操作执行多次和执行一次，结果是一样的。例如，5*1 = 5*1*1*1，所以对5的乘1操作具有幂等性。日志文件在故障恢复中，可能会回放多次（比如第一次回放到一半时系统断电了，不得不再重新回放），如果操作记录不满足幂等性，会造成数据错误。

资料
What Is Undo?

Redo log

Oracle学习笔记：Redo日志(重做日志)的作用 

数据库日志文件– undo log 、redo log、 undo/redo log 

undo log与redo log原理分析

Oracle redo与undo浅析

RedoLog Checkpoint 和 SCN关系 

mysql dba系统学习（10）innodb引擎的redo log日志的原理

2.4　Checkpoint技术

MySQL Innodb日志机制深入分析

《MySQL技术内幕：InnoDB存储引擎（第2版）》

本文作者： 樂天

本文链接： http://www.letiantian.me/2014-06-18-db-undo-redo-checkpoint/

版权声明： 本博客所有文章除特别声明外，均采用 CC BY-NC-SA 3.0 许可协议。转载请注明出处！

