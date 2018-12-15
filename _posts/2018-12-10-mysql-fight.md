---
layout: post
title: 'mysql实战'
subtitle: 'mysql实战'
date: 2018-12-10
categories: mysql
author: yates
cover: ''
tags: mysql
---


## mysql逻辑架构

**mysql处理流程**
客户端->连接器->分析器->查询缓存->优化器->执行器->存储引擎

整个结构大致可分为**server层**和**存储层**，不同的存储引擎使用同一个server层

**存储引擎**
作用:**存储数据**，提供**读写接口**;

**连接器**
作用: **管理连接，权限验证**,我们经常使用(mysql -h ip -P port -u user -p pw)指令来进行server的连接 这一步不通过的会提示**Access denied for user** 的错误

一个用户如果建立连接后，即使管理员账户对这个用户权限做了修改，也不会影响已经存在的连接的权限，修改完成后新建的连接才会使用新的权限设置

(**show processlist**)指令查询连接用户的信息和状态

客户端如果太长时间没有动静，连接器会自动断开，这个时间由wait_timeout控制，默认8小时，连接被断开后会收到**lost connection to mysql server during query**的错误

**长连接**：连接成功后，客户端持续有请求，则一直使用同一个连接
**短连接**：每次执行完很少的几次查询就断开连接，下次查询再重新建立一个

建立连接的过程通常比较复杂，建议使用长连接，但是长连接会**导致mysql占用内存涨得特别快**，由于sql执行中的使用**临时内存**是管理**连接对象**里面的，最终导致为了释放内存，mysql会被异常重启，解决方案（1定期断开长连接 2 5.7版本后每次执行较大操作后，通过mysql——reset_connection重新初始化连接资源，而且不用重连和重新做权限验证）

**查询缓存** 
查询请求会被以**key-value**存储在内存中，key是查询语句，value是查询结果，如果在查询中
命中则直接返回结果.建议不要使用查询缓存，如果必要使用query_cache_type设置成DEMAND进行按需获取缓存，select SQL_CACHE * FROM T WHERE ID =10; MYSQL8后完全删除了查询缓存模块

**分析器**
作用: **词法分析，语法分析**.通过词法和语法分析识别语句的**类型，解析表，字段，验证语法**规则等操作，这一步不通过的会提示(**sql syntax**)的错误

**优化器**
作用：**执行计划生成，索引选择**.优化器进行索引的选择，多表连接时表连接顺序的选择等操作

**执行器**
作用：**操作引擎，返回结果**.开始执行时，要先判断你对操作表有没有执行查询的**权限**，若没有会返回(**select command denied to user ** for table ***).然后打开表，根据表的引擎定义，使用引擎提供的接口，进行表数据一行一行的判断和读取，在慢查日志汇总有一个row_wxamined字段，表示这个语句扫描了多少行，但是这个并不完全正确的。



## myql数据库的数据更新技术-WAL
再生产中,想恢复某个数据时间点的数据,除了通过备份数据外,还可通过redo log和binlog进行重放

**redo log 和binlog**
mysql更新操作采用**WAL(write-ahead logging)**技术，使用 **redo log**，先将更新写入日志，空闲时在写入磁盘，表示记录这个数据页做了什么改动
Innodb的redo log是一组4个文件，每个文件1GB大小空间，有两个标志位**wirte pos**和**checkpoint**，

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/1.png) 

write pos表示当前记录的位置，checkpoint表示当前要擦除的位置，wirite pos不断的向后移动写入redo log，
checkpoint的不断将rodo log数据写入磁盘，当writelog追上checkpoint后，就要等待checkpoint清理出一片空间后才能继续写入
从而保证了数据库发生异常重启，之前的记录都不会消失，被称为**crash-safe**，redo log作为引擎层特有日志

innodb_flush_log_at_trx_commit 设置成1，表示每次事务的redo log都直接持久化到磁盘。

**server层由binlog做归档日志**,记录所有的逻辑操作，所有引擎都可以使用，以前写的日志不会覆盖，写在**redolog pre 和redolog commit**之间，两阶段提交能够保证了数据存储的有效性。binlog有两种模式，**记sql语句和记录行行内容前后的状态**

**数据库恢复**：
1首先找到最近的一次全量备份
2从上次开发备份时间开始，将备份的binlog依次取出来，进行重放。

sync_binlog 参数设置为1 表示每次事务的binlog都持久化到磁盘，保证mysql异常重启后binlog不丢失。

## 数据库隔离级别实现

未提交读，直接**返回记录**上的最新值

提交读和可重复读是使用**视图**来访问数据，**可重复读**是在**事务启动的时候创建**的，**读提交**级别是在**每个sql启动时创建**的。

串行化时直接进行**加锁**。

每条记录在更新的时候会同时记录一条回滚操作，**不同时刻启动的事务会有不同的read-view**，在不同view里面一条记录有不同的值，这就是数据库多版本控制(MVCC).
当**系统中没有比回滚日志更早的view**时，日志就会删除，长事务会保存很老的事务视图，占用大量的存储空间。

## 事务启动方式

1. set autocommit=0，关闭自动提交事务，然后使用commit提交事务  
2. 显式使用 begin/start transaction语句

**查询大于60s的长事务**

```java
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

## 索引搜索模型

**常见索引类型**
**哈希表**：通过链表结构存储索引数据，等值搜索很快，但是区间查找效率就不高
**有序数组**：等值和，区间查找效率都很高，但是不利于数据的更新，插入，删除记录会挪动其他数据，成本太高，只适合用于静态存储引擎。
**搜索树**：每一个索引在innodb里面对应一棵B+树。

**索引的工作原理以及相关知识**

- 主键索引被称为聚簇索引，其叶子节点存的是整行数据
- 非主键索引称为二级索引，其叶子节点存储的是主键的值,非主键索引查询方式，先搜索非主键索引树，通过拿到主键进行主键索引树的搜索（称为**回表**）。
- 因为非主键索引会存储主键索引的值，所以如果主键索引越长，那么非主键索引叶子节点占用的空间就越大，而数据占用空间膨胀会导致**页分裂**。索引一般提倡使用自增主键。（在只有一个索引，而且该索引为唯一索引的情况下，业务字段是适合做主键的）
- 由于非主键索引k的叶子节点上面已经有主键的id值，所以如 select id from table where k bettwen 2 and 6 等sql语句时，索引K以及覆盖了我们的查询需求，我们称为**覆盖索引**，覆盖索引可以减少数据搜索次数
- 对于联合索引（a,b），针对a的索引可以复用，可能会需要另外建立b的索引，那么a的如果比b的占用空间大，在建立b单独索引时就会节省空间

- **索引下推**，5.6版本以后引入的索引下推，在索引遍历的过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数

- 重建主键索引（删除，创建）都会将整个表重建，而且这个表上的其他非主键索引也会失效，可以使用alter table engine = innodb代替


## 全局锁，表级锁，行级锁
### 全局锁实现方式 

- sql语句:  **Flush tables with read lock**;常用于全库逻辑备份，备份时不加锁会造成多表操作时事务的不一致性，比如转账，购买业务场景
- sql语句:  **set global readonly=true**（如果执行过程中客户端断开，数据库会一直保持**readonly**状态，而**ftwrl**不会，所以该语句通常用于**判断主从库**的逻辑）
- mysql自带的mysqldump逻辑备份工具: 但是只适用于使用事务引擎的库表 （–single-transaction参数）

### 表级锁实现方式 

- sql语句: **lock tables T read/write**，除了限制别的线程读写外，也限定了本线程接下来的操作对象;   比如:（thread1 lock table1 read，table2 write；那么thread2 对 t1，t2读写都不可进行；自身线程不能对T1做write操作）
- MDL:  访问表时自动加上，mysql5.5以后在对一个表做**增删改查时，加MDL读锁**；当对**结构做变更操作时，加MDL写锁**。

**注意**

- 读锁之间不互斥
- 读写锁和写锁之间互斥

在**热点查询表**中执行**DDL**，很有可能造成整个数据库挂掉，原因是**执行DDL线程时会阻塞后面的请求**，而如果DDL线程前面的请求一直**不释放MDL锁**的话，客户端的不断重试，生成新session，数据库线程爆满。这种情况在做DDL时，先将长事务kill掉
事务中的 MDL 锁，在语句执行开始时申请，但是语句结束后并不会马上释放，而会等到整个事务提交后再释放。

**解锁**：
unlock tables

### 行级锁实现方式 

**两阶段锁协议**
 在innodb事务中,行锁是**在需要的时候才加上**的,但并不是不需要就立刻释放,而是**等待事务结束后才释放**.基于这个原理,在一个事务处理中,合理安排**sql执行顺序**减少事务之间的**锁等待**,能够有效的提高**并发度**
 
**死锁**
死锁的产生是由于**事务之间的资源调用互相占用**造成的,虽然可以通过**执行相同的资源调用顺序**来规避,但是我们在平时的开发中去记住各个资源的调用顺序是不现实的.通常使用两种策略:

1. 通过**设置等待超时**来释放资源(**innodb_lock_wait_timeout**)
2. 发起**死锁检测**,发现死锁后,主动回滚死锁链条中的某个事务((**innodb_deadlock_detect = on(**)

这两种方法,前者超时时间设置**长了会造成资源长期得不到利用**;设置**短了很容易造成长事务的误伤**;后者是经常使用到的方式,但是也有缺点.线程在**检测死锁**时,需要去**循环所有依赖线程**,判断**自己是否加入导致死锁**,这个时间复杂度为**O(n)**,当并发上去以后,会导致**cpu大量用于检测死锁占用**.

那么怎么**解决**这种问题的发生呢?

1. 如果保证确保这个业务不会出现死锁,可以**临时关闭死锁检测**,但是会**导致大量的超时**,对业务有损
2. **控制并发度**,核心是**确保单行的更新线程**很少,这个并发度在哪里做的问题,就要具体问题具体分析了,一般**服务端**可以做,**中间件**也可以做,**数据库库服务端**也可以做,还有一个方案是**单行数据拆分为多行数据**,比如说某个账户拆分为N个子账户,需要进行更新时随机进行一条记录操作,但是要做好**业务逻辑**上的处理


## 快照在MVCC里的生成
Innodb 里面**每个事务**有一个**唯一的事务ID**,叫作**trancactionid**,事务开始的时候生成,且**递增**,**每行数据也有数据版本**,每次事务更新数据时,都会生成新的数据版本,并且把transaction id赋值给这个数据版本的事务ID,记为**row_trx_id**,旧版本也会直接保存在新版本中

如图: 版本V1到V4的trans id变化,图中**虚线部分**就是redo log日志,而v1到v3物理上是不存在的,是通过redo log计算出来的

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/2.png)

可重复读的定义是当一个**事务启动时,只会认可在此之前的数据版本**.Innodb为**每个事务**构造一个**数组**,用来保存所有**正在活跃(启动了没有提交)的事务ID**.
数组里面事务ID最小值记为**低水位**,当前系统里面已经创建的事务ID最大值+1称为**高水位**,视图数组和高水位,构成了当前事务的**一致性视图**

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/3.png)

当前事务启动瞬间,一个数据版本row_trx_id,有可能分布在上述绿,黄,红色部分中,如果在**绿色:对于当前事务数据是可见**的.如果在**红色部分,这个版本时是来事务启动的,不可见**,如果**黄色部分而且row_tran_id在数组中,表示没提交事务生成,不可见;否则可见**

如图
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/4.png)

事务A读到的数据k值为1,那么事务B读到的值是1,但是更新操作得到的数据确实3,因为对于事务B来说,**update操作都是先读后写**的,而这个读只能读取当前的值,称为"**当前读**".如果**select语句加锁**的话也会"当前读".假设事务C的提交在事务B的更新之后,那么事务B更新时读到的K又是多少了,答案是两阶段锁协议会导致事务B阻塞等待事务C的完成.

总结:可重复读的核心就是**一致性读（consistentread）**;而**事务更新数据的时候，只能用当前读**。如果当前的记录的**行锁被其他事务占用**的话，就需要进入**锁等待**。

而读提交和可重复读区别在于,前者是在**每一个语句之前**重新计算一个**新的一致性视图**,后者是在**事务开始时创建**,并在**事务范围内其他查询共用**.

**唯一索引和普通索引的查询**
Innodb的数据是按照数据页为单位来读写的,也就是说读取一条记录的时,以页为单位将其整体读入内存,唯一索引和普通索引的查询成本相当

**唯一索引和普通索引的更新**
**change buffer**

当需要更新一个数据页时,如果数据页在内存中就直接更新,如果不在,Innodb会将这些更新操作缓存在change buffer中,这样不需要从磁盘中读取数据页,下次有关这个数据页的操作时,将数据页读入内存,执行change buffer的操作,这个过程称为merge.除了访问数据页会触发,后台也会定期merge.change buffer在内存中有拷贝,也会被写入到磁盘.

唯一索引在插入数据时,都会去判断是否唯一,如果刚好插入页在内存中,则和普通索引没有区别;若不是则要去磁盘进行校验,相比普通索引则最多借助change buffer的帮助进行数据更新

change buffer只适用于普通索引的场景,且写多读少的业务,对于写入后马上就会进行查询的业务,需先记录changeBufer,然后merge,增加了change buffer的维护成本.

redolog在更新数据的时候会写入.

**redolog主要是节省随机写磁盘的IO消耗,而change buffer主要节省的是随机读磁盘的IO消耗**


## 索引选择
mysql也会有**选错索引**的情况特别是在平常不断**删除历史**和**新增数据**的场景下,优化器在评估使用什么索引的情况下,首先是通过**预估扫描行数**,然后评估**是否回表**等等综合考虑

**扫描行数预估**
通过**索引区分度**来计算,一个**索引**上的**值越多**,**区分度越高**,这个值称为**基数**.

**查看区分度** (**show indes from t**)中的**cardinarty**项,这个项mysql通过**采样统计**,先获取一个页的索引基数,然后乘以页数

**回表因素**
如果我们有一个索引B 和主键索引id,前一个查询扫描行有1000条,后者需要10000条,又是mysql会选择后者,因为mysql会觉得我10000条直接扫主键,不用回表,没有额外的代价

**重建索引**
大多数情况先如果我们发现msyql分析出来的**扫描数**和实际的**差距过大**,可以通过指令 (**analyze table t**)来重建索引,使mysql得到正确的扫描数


### 极少情况选错索引的修正

- 强制使用指定索引.**多索引**的情况下,极少情况会出现选错索引的情况,碰到这种情况,如果想要修正异常,可以使用 **force index(a)** 来**强制**使用**指定索引**

- **修改sql语句**,引导mysql使用我们期望的索引.

- 新建**更合适**的索引或者**删除误用**的索引.


**字符串怎么创建索引**
- 前缀索引,









