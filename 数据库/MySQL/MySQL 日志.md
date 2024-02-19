> [MySQL 日志：undo log、redo log、binlog 有什么用？ | 小林coding (xiaolincoding.com)](https://www.xiaolincoding.com/mysql/log/how_update.html)
> 
> 【动画讲解：MySQL两大内存BufferPool、RedoLogBuffer和三大日志binlog、redolog和undolog】 https://www.bilibili.com/video/BV1Dr4y197ai/?share_source=copy_web&vd_source=02e1fcde003194e9ed62527f120b831f
> 
> [一文搞懂MySQL各种日志-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2339983)
> 
> [MySQL中都有哪些日志文件？_数据库日志文件中包含哪些内容-CSDN博客](https://blog.csdn.net/lveex/article/details/118873638)


## MySQL 三大日志 undo log、redo log、binlog 有什么用
- **undo log（回滚日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**原子性**，主要**用于事务回滚和 MVCC**。
- **redo log（重做日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**持久性和一致性**，主要**用于掉电等故障恢复**；
- **binlog （归档日志）**：是 Server 层生成的日志，主要**用于数据备份和主从复制**；
### MySQL 系统日志架构设计
![](http://douyin.cfddfc.online/myPicture/20240131223701.png)
_图片来自[动画讲解：MySQL两大内存BufferPool、RedoLogBuffer和三大日志binlog、redolog和undolog_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Dr4y197ai/?spm_id_from=pageDriver&vd_source=2d885cb62bb9393fa8a5379c72eabd82)_
### undo log
**undo log（回滚日志），它保证了事务的 ACID 特性中的原子性（Atomicity）**。

undo log 是一种用于撤销回退的日志。在事务没提交之前，MySQL 会先**记录更新前的数据**到 undo log 日志文件里面，当事务回滚时，可以利用 undo log 来进行回滚。

在发生回滚时，读取 undo log 里的数据，然后做**原先相反操作**，**将数据恢复到事务开始之前的状态**。

> **MVCC**：事务未提交之前，Undo Log保存了未提交之前的版本数据，Undo Log中的数据可作为数据旧版本快照供其他并发事务进行**快照读**。事务A手动开启事务，执行更新操作，首先会把更新命中的数据备份到 Undo Buffer 中。事务B手动开启事务，执行查询操作，会读取 Undo 日志数据返回，进行快照读。
> ![](https://img-blog.csdnimg.cn/20210718104951377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hzZjE1NzY4NjE1Mjg0,size_16,color_FFFFFF,t_70)
### redo Log
#### 概念
用于恢复数据库在发生故障时的**持久性和一致性**状态，可以在崩溃修复期间纠正不完整事务写入的数据。

**当事务提交时，它的修改会首先写入redo log，然后再写入数据库文件。**

在数据库崩溃后，通过重放redo log中的操作，可以将数据库还原到最后一次提交的状态，确保数据的**持久性和一致性**。
#### 原理
**Undo Log是在事务开启的时候产生，而Redo Log是在事务执行的过程产生。**

在事务提交时会将产生Redo Log写入Log Buffer，但并不是随着事务的提交就立刻写入磁盘，而是等到事务操作的脏页写入到磁盘之后，Redo Log占用的空间就可以重用（被覆盖写入）。

Redo Log文件内容是以顺序循环的方式写入文件，写满时则回溯到第一个文件，进行覆盖写。
![](https://img-blog.csdnimg.cn/20210718111840875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hzZjE1NzY4NjE1Mjg0,size_16,color_FFFFFF,t_70#pic_center)
如上图所示，**Redo Log采用双指针进行维护。** Write Pos是当前记录的位置，一边写一边后移，写到最后一个文件末尾后就回到0号文件开头；CheckPoint是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件；

Write Pos和CheckPoint之间还空着的部分，可以用来记录新的操作。如果Write Pos追上CheckPoint，表示写满，这时候不能再执行新的更新，会停下来先擦掉一些记录。如图：W区则是可以写入的区域，R区则是需要刷盘的内容。
### binlog
记录数据库的**所有 修改 操作**，包括对数据的**增删改操作**，以及对**表结构的修改**。

Binlog主要用于实现数据库的**复制**、**恢复**和增量备份。

通过分析Binlog，可以重放数据库的修改操作，实现主从复制和其他数据复制方案。

> **复制：** 目前我们常说的高可用/高并发，那数据库的高可用/高并发就可以通过各个数据库实例之间复制binlog以达到数据的同步。**如mysql的主从复制**
> 
> **恢复：** 某些数据可以通过执行binlog来恢复。
## 其他日志
### 错误日志（error log）
记录MySQL 运行过程中较为严重的警告和错误信息，以及MySQL每次启动和关闭的详细信息，默认是开启的。

通过`show variables like '%log_error%'`查看。
### 中继日志（relay log）
中继日志主要是MySQL**主从同步**的时候会用到，在从库中担当类似中转的作用。

主从同步主要分成三步：
1：主库将数据库的变更操作记录到Binlog日志文件中
2：从库读取主库中的Binlog日志文件信息写入到从库的Relay Log中继日志中
3：从库读取中继日志信息在从库中进行Replay，更新从库数据信息

> 更具体的主从复制可以看这个[这次终于把MySQL主从复制总结全面了！！！-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1822790)
> 1. master 服务器会将 SQL 记录通过多 dump 线程写入到 binary log 中。
> 2. slave 服务器开启一个 io thread 线程向服务器发送请求，向 master 服务器请求 binary log。master 服务器在接收到请求之后，根据偏移量将新的 binary log 发送给 slave 服务器。
> 3. slave 服务器收到新的 binary log 之后，写入到自身的 relay log 中，这就是所谓的中继日志。
> 4. slave 服务器，单独开启一个 sql thread 读取 relay log 之后，写入到自身数据中。

#### **为什么从库读取主库中的Binlog日志为什么不直接执行，而是先写入到Relay Log后面由读出来？**

1. **异步复制：** 主从复制通常是异步进行的，即从库的更新不需要等待主库执行完毕。直接执行主库的Binlog日志可能导致从库执行速度不如主库，从而导致复制延迟。通过先将主库的Binlog日志写入Relay Log，可以在从库自行决定何时以何种速度执行这些日志，使得从库的执行速度不受主库的限制。
    
2. **执行的灵活性：** 将主库的Binlog日志写入Relay Log后，从库可以在执行之前进行一些额外的处理，例如过滤不需要的操作、改变执行顺序等。这为从库提供了更大的灵活性，可以根据实际需求对复制的行为进行定制。
    
3. **缓冲和存储：** 将主库的Binlog日志写入Relay Log可以起到缓冲的作用，即使从库暂时失去与主库的连接，也能够在重新连接后继续执行。此外，Relay Log的使用也允许从库存储其自己的二进制日志，这样就可以再次被其他从库用作主库进行级联复制。
    
4. **安全性考虑：** 将主库的Binlog日志写入Relay Log提供了一种更为安全的方式，从库可以在写入Relay Log之前对Binlog进行一些检查和验证，确保复制的数据的完整性和正确性。
    

总的来说，通过引入Relay Log，MySQL的主从复制系统更具有灵活性、可定制性和容错性，从而适应不同的应用场景和需求。

### 慢查询日志
慢查询日志用于记录执行时间超过一定阈值的查询语句。**通过分析慢查询日志，可以识别执行时间较长的查询。**
1. **启用和配置：**
   - 默认情况下，慢查询日志是禁用的。要启用慢查询日志，需要在MySQL的配置文件中进行相应的配置。
   - 通过设置慢查询的时间阈值（`long_query_time`），可以定义哪些查询被认为是慢查询。
   - 可以通过`show variables like '%slow_query%'`查看是否开启和日志的位置

3. **分析工具：**
   - MySQL提供了一些工具用于分析慢查询日志，如`mysqldumpslow`和`pt-query-digest`。这些工具可以帮助用户汇总和分析慢查询日志，找出执行时间最长的查询。

4. **用途和优势：**
   - **性能优化：** 通过检查慢查询日志，可以识别哪些查询语句执行时间过长，从而进行性能优化，优化的方式可能包括索引的添加、查询语句的调整等。
   - **故障排查：** 慢查询日志也可用于故障排查，例如查找某些查询语句导致的**锁等待或死锁**情况。
   - **合理设置阈值：** 设置适当的慢查询时间阈值对于平衡性能分析和减少日志量至关重要。如果设置得太低，可能会记录过多的查询，占用过多磁盘空间。

5. **安全性：**
   - 由于慢查询日志可能包含敏感信息，如密码等，因此在生产环境中需要谨慎处理日志文件的访问权限和存储位置。
