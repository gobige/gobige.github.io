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
在生产中,想恢复某个数据时间点的数据,除了通过备份数据外,还可通过redo log和binlog进行重放

**redo log 和binlog**
mysql更新操作采用**WAL(write-ahead logging)**技术，使用 **redo log**，先将更新写入日志，空闲时在写入磁盘，表示记录这个数据页做了什么改动
Innodb的redo log是一组4个文件，每个文件1GB大小空间，有两个标志位**wirte pos**和**checkpoint**，

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/1.png) 

write pos表示当前记录的位置，checkpoint表示当前要擦除的位置，wirite pos不断的向后移动写入redo log，
checkpoint的不断将rodo log数据写入磁盘，当writelog追上checkpoint后，就要等待checkpoint清理出一片空间后才能继续写入
从而保证了数据库发生异常重启，之前的记录都不会消失，被称为**crash-safe**，redo log作为引擎层特有日志

innodb_flush_log_at_trx_commit 设置成1，表示每次事务的redo log都直接持久化到磁盘。

**server层由binlog做归档日志**,记录所有的逻辑操作，所有引擎都可以使用，以前写的日志不会覆盖，写在**redolog pre 和redolog commit**之间，两阶段提交能够保证了数据存储的有效性。binlog有两种模式，**记sql语句和记录行行内容前后的状态**

**redo log和bin log写入发生异常及恢复**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/5.png)

如图:一条数据在写入的时候binlog和redolog的写入过程
- 如果mysql奔溃发生A时刻,在redolog prepare阶段,写binlog之前,那么奔溃恢复时事务自然会回滚
- 如果mysql奔溃发生在B时刻,在binlog写完,redolog还没有提交时,那么通过以下规则判断
	- 如果redolog事务完整,已经有了commit标识,那么直接提交
	-  如果redolog没有提交只有完整prepare,若binlog如果完整,则提交;反之binlog不完整则回滚

那么mysql通过什么值得binlog是否完整的呢?

- statement的binlog,最后会有commit;
- row格式的binlog,最后会有一个XID event. 
- mysql5.6.2版本后还引入了binlog-checksum参数,用于验证内容正确性

redolog和binlog通过什么关联?通过XID标识关联,binlog在写入后就会从从库取出来,所以主库也要通过redolog提交这个事务,从而保证数据一致性.


**数据库恢复**：

1. 首先找到最近的一次全量备份
2. 从上次开发备份时间开始，将备份的binlog依次取出来，进行重放。

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

- **主键索引**被称为**聚簇索引**，其**叶子节点**存的是**整行数据**
- **非主键索引**称为二级索引，其**叶子节点**存储的是**主键的值**,非主键索引查询方式，先搜索非主键索引树，通过拿到主键进行主键索引树的搜索（称为**回表**）。
- 因为非主键索引会存储主键索引的值，所以如果**主键索引越长**，那么**非主键索引叶子节点**占用的**空间越大**，而数据占用空间膨胀会导致**页分裂**。索引一般提倡使用自增主键。（在只有一个索引，而且该索引为唯一索引的情况下，业务字段是适合做主键的）
- 由于非主键索引k的叶子节点上面已经有主键的id值，所以如 select id from table where k bettwen 2 and 6 等sql语句时，索引K以及覆盖了我们的查询需求，我们称为**覆盖索引**，覆盖索引可以减少数据搜索次数
- 对于**联合索引**（a,b），针对a的索引可以复用，可能会需要另外建立b的索引，那么a的如果比b的占用空间大，在建立b单独索引时就会节省空间

- **索引下推**，5.6版本以后引入的索引下推，在索引遍历的过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数

- **重建主键索引**（删除，创建）都会将整个表重建，而且这个表上的其他非主键索引也会失效，可以使用**alter table engine = innodb**代替


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

可重复读级别在事务启动时生成的视图是整个数据库数据的视图，俗称快照，通过事务ID实现

Innodb 里面**每个事务**有一个**唯一的事务ID**,叫作**trancactionid**,事务开始的时候生成,且**递增**,**每行数据也有数据版本**,每次事务更新数据时,都会生成新的数据版本,并且把transaction id赋值给这个数据版本的事务ID,记为**row_trx_id**,旧版本也会直接保存在新版本中

如图: 版本V1到V4的trans id变化,图中**虚线部分**就是redo log日志,而v1到v3物理上是不存在的,是通过redo log计算出来的

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/2.png)

可重复读的定义是当一个**事务启动时,只会认可在此之前的数据版本**.Innodb为**每个事务**构造**一个视图数组**,用来保存所有**正在活跃(启动了没有提交)的事务ID**.
数组里面事务ID最小值记为**低水位**,当前系统里面已经创建的事务ID最大值+1称为**高水位**,视图数组和高水位,构成了当前事务的**一致性视图**

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/3.png)

当前事务启动瞬间,一个数据版本row_trx_id,有可能分布在上述绿,黄,红色部分中,如果在**绿色:对于当前事务数据是可见**的.如果在**红色部分,这个版本时是来事务启动的,不可见**,如果**黄色部分而且row_tran_id在数组中,表示没提交事务生成,不可见;否则可见（比低水位大，但是在当前事务启动前，就已经提交了）**

如图
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/4.png)

事务A读到的数据k值为1,那么事务B读到的值是1,但是更新操作得到的数据确实3,因为对于事务B来说,**update操作都是先读后写**的,而这个读只能读取当前的值,称为"**当前读**".如果**select语句加锁**的话也会"当前读".假设事务C的提交在事务B的更新之后,那么事务B更新时读到的K又是多少了,答案是两阶段锁协议会导致事务B阻塞等待事务C的完成.

总结:可重复读的核心就是**一致性读（consistentread）**;而**事务更新数据的时候，只能用当前读**。如果当前的记录的**行锁被其他事务占用**的话，就需要进入**锁等待**。

而读提交和可重复读区别在于,前者是在**每一个语句之前**重新计算一个**新的一致性视图**,后者是在**事务开始时创建**,并在**事务范围内其他查询共用**.

**唯一索引和普通索引的查询**
Innodb的数据是按照数据页为单位来读写的,也就是说读取一条记录的时,以页为单位将其整体读入内存,唯一索引和普通索引的查询成本相当

**唯一索引和普通索引的更新**

当需要更新一个数据页时,如果数据页**存在内存**中就**直接更新**,如果**不在**,Innodb会将这些更新**操作缓存在change buffer**中,这样不需要从磁盘中读取数据页,**下次有关这个数据页的操作**时,将**数据页读入内存**,**执行change buffer**的操作,这个过程称为**merge**.除了访问数据页会触发,后台也会定期merge.change buffer在内存中有拷贝,也会被写入到磁盘.

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


### 字符串怎么创建索引 

**hash和前缀索引**
- hash索引,直接从索引树找到满足索引值的记录,回表查找相应主键,返回记录,继续索引树下一条记录对比,直到找到非条件的记录
- 前缀索引,过程跟hash差不多,只是由于索引树存储的字符串部分数据,会造成回表的次数变多.

可以看出前缀索引唯一优势就是节省了**索引的存储空间**,所以定义好前缀的长度就显得尤其重要了,通过sql(**select count(distinct column) as L from table;**)查看某字段不同的值的数量,通常设计前缀索引所造成的**损失区分度**在5个百分点之内.

使用前缀索引会导致覆盖索引的查询性能优化失效.


**倒序存储**
使用在一些后面位数区分度高一些的字符串,例如 存储身份证号的时候把它倒过来存，每次查询的时候(**select field_list from t where id_card = reverse('input_id_card_string');**) 

**使用hash字段**
在表上再创建一个整数字段,来保存hash过的字符串校验码(**mysql> alter table t add id_card_crc int unsigned, add index(id_card_crc);**)

倒序和使用hash字段都不支持范围查询

### mysql抖动
很懂场景下,你会发现,一条sql语句,正常情况下执行会特别快,但是有时会特别慢,这种发生时随机的,持续时间很短.大多数情况是由于mysql在刷**脏页**

**脏页**
前面讲到,innodb在处理更新语句的时候,会先记录redo log,随后有空再将redo log的更新到磁盘,称为 **flush**.那么我们称,**内存数据页**和**磁盘数据页**内容不一致的时候,这个内存页称为**脏页**,内存经过flush到磁盘后,内存数据页和磁盘数据页内容一致后,称为 **干净页**.

**flush的场景**

- redo log写满了,write pos无法推进,需要暂时暂停,将checkpoint向前推进,给redo log留出可用空间
- 系统内存不足,当需要新的内存页的时候
- mysql认为系统空闲的时候
- mysql正常关闭的时候

**innodb刷脏页的策略控制**
innodb刷脏页的能力,决定在于主机的磁盘读写能力,通过 innodb_io_capacity参数的设置成主键的磁盘IOPS.(fio工具可以测试出IOPS)

**表数据的存储**
表数据可以存在共享表空间,也可以是单独的文件.
**innodb_file_per_table=OFF**:表数据放在**系统共享表空间**,也就是跟数据字典一起

**innodb_file_per_table=ON**:表数据存储在一个以**.ibd**为后缀的文件中.

**数据记录和数据页的可复用状态**
有时我们会发现我们使用delete语句删除了数据之后,发现表空间大小没有变化,这是由于delete语句只是将删除的记录置为**可复用**状态,磁盘文件大小并不会缩小,看起来就像是**空洞**,不仅删除会造成空洞,插入数据也会,如果数据不是按照索引递增顺序插入,而是随机插入的,就可能造成索引的**数据页分裂**.

**重建表**
为了使索引数据页看起来更紧凑,可以通过语句(**alter table A engine=InnoDB**)重建表.5.6之后引入了Online DDL,支持在表重建过程中保证**新数据的写入安全**

**统计表的记录**
Myasiam引擎把一个表的总行数存在了磁盘上,执行count(*)直接返回这个数
Innodb因为支持多事务MVCC版本控制所以需要一行一行从引擎读出来,累计计数.

innodb在执行 count(#) 时优化器会选择**最小的那颗索引树**来进行遍历. **show table status**语句也可统计行数,但是这个语句统计行数是通过采样方式计算出来的(数据页记录*页数),统计出来的数据是不准确的

count(id),count(col),count(1)等有不同的性能,在分析性能差别时,有几个原则:
1. server要什么给什么
2. Innodb只给必要的值
3. 现在优化器只优化了count(*)的语义为 "取行数",其他显而易见的优化并没有做

**count(id)**:innodb遍历整张表,把每一行id取出来,返回server,server判断id不为空的,按行累加

**count(1)**:遍历整张表,但不取值,对于返回每一行,放一个数字 1 ,按行累加.

**count(col)**:如果字段允许为null,执行的时候,还需要把值取出来再判断一下,不是null才累加

按照效率排序的话，count(字段)<count(id)<count(1)~count(*)

**其他统计记录的方式**
把计数放redis里面:由于**两个不同的存储构成的系统,不支持分布式事务,无法拿到精确一致的视图**.不能保证计数和mysql表里数据精确一致.


**order by实现流程**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/6.png)
上述mysql排序可能在**内存**中进行,也可能会使用**外部排序**,取决于 sort_buffer_size设置的大小,如果需要排序的数据小于这个设置值,则在内存中进行;大于则将数据分成该设置值大小的外部文件,分别排序,最后合并

 
检查排序是否使用了临时文件
```java
/* 打开 optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 

/* @a 保存 Innodb_rows_read 的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 执行语句 */
select city, name,age from t where city='杭州' order by name limit 1000; 

/* 查看 OPTIMIZER_TRACE 输出 */
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G

/* @b 保存 Innodb_rows_read 的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 计算 Innodb_rows_read 差值 */
select @b-@a;

```


**如果排序返回字段过多**
如果排序后返回的字段很多,会导致,排序文件和内存占用过大,那么就要使用其他算法.
```java
SET max_length_for_sort_data = 16;
```
我们设置了排序的行长度不能超过16,如图,上述返回的字段超过了16,那么放入sort_buffer的字段就只有name和主键id,流程变成如下:
1. 初始化sort_buffer,放入两个字段,name和id
2. 从索引city取出第一个满足条件的主键id
3. 通过主键id回表取出整行,取name,id字段,存入sort_buffer
4. 从索引取符合条件的下一个主键id
5. 重复3,4步骤
6. 对sort_buffer数据按name排序
7. 遍历排序,取前1000行
8. 再次回表通过id取出其他字段数据

这个过程称为**rowid排序**,其中8不需要服务端再存入内存,而是直接返回给客户端

**如果内存够，就要多利用内存，尽量减少磁盘访问,采用全字段排序,否则使用rowid排序**

**所有order by都会排序是mysql使用排序算法吗**
如果我们建一个联合索引,如(city,name)那么上述的sql后面应用索引得到4步骤数据就已经是排好序的数据了

**取随机数**
mysql中我们使用rand()获取随机数,他的流程如下:
1. 创建临时表,使用功能memory引擎,表里有两个字段,一个是R,一个W.这个表没有索引
2. 按主键顺序取出所有word,调用rand()函数生成一个大于0小于1的随机小数,将随机小数和word存入R和W中.
3. 在临时表上按照字段R排序.
4. 初始化sort_buffer,sort_buffer中也有两个字段,一个是double,另一个整形.
5. 将临时表一行一行取出R和**位置信息**放入sort_buffer中
6. 在sort_buffer中根据R排序,并取出所需数据位置信息

流程如如图:
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/7.png)

**先通过原理分析出扫描行数,再通过慢查日志,验证结论,是一种很好的方法**

**mysql定位一行数据的方式**

- 对于有主键的innodb表,主键就是定位方式.
- 对于没有主键的innodb表,自动生成的rowid就是定位方式.
- 针对memory引擎的组织表,可以认为就是一个数组,数组下标就是定位方式

我们把这种定位方式成为**rowId排序**,也就是上述的orderBy使用到的方式,而在rand函数执行,内存临时表排序时也使用了rowid排序方法

临时表也可以使用**磁盘**,tmp_table_size限制内存临时表大小,默认为16M.

有时候我们使用rand函数后后面还跟了个limit N,而这时mysql不会使用磁盘的归并排序,因为我们只需limit数量的数据,所以排序所有的数据不是mysql所希望的,这时采用**优先队列排序**,以N行数据为一个单位,拿单位边界外数据与单位内数据进行对比,一个遍历后单位内数据是我们所需得到的limit数据.(limit数据如果过于大,超过sort_buffer_size大小的话,就只能使用归并排序算法).

**更具效率的排序方式**
算法1

- 取这个表的主键id最大值M和最小值N
- 用随机函数生成一个最大值到最小值之间的数X=(M-N)*rand()+N
- 取不小于X的第一个ID的行

这个算法有个bug,就是如果id不连续,很有可能会得到一个**空洞**的数据

算法2

- 取这个表的行数,记为C
- 取得Y = floor(c * rand()) 
- 再用Y,1取得一行数据


算法3

- 取整个表行数 C
- 根据随机方法得到Y1,Y2,Y3
- 在执行三次limit Y,1得到三行数据


### 单行sql哪些情况下会变慢

**函数的使用**

条件字段**函数的使用**会导致索引字段的有序性遭到破坏，于是优化器在sql优化时会放弃走该字段的树搜索功能，造成sql语句执行慢

**数据类型转换**

数据类型转换规则：字符串和数字作比较的话，是将**字符串转换成数字**。因此mysql中隐式转换也会造成sql语句执行慢

**隐式字符编码**

**隐式字符编码**转换，如果两种表的字符集使用不统一，那么在两张变做关联查询时，被驱动表的索引字段有可能作隐式转换，从而导致对被驱动表作全表扫描，sql执行慢。解决方案（1统一编码字符集，2通过修改关联表之间连接顺序）

**锁表**

通过show processlist命令，查看当前语句的状态，如果看到**Waiting for table metadata lock**的状态，说明存在线程正在表T上请求或持有**MDL写锁**，该查询语句被阻塞,如图：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/8.png)
- 针对这种阻塞的处理方式时找到持有MDL写锁的线程，然后kill掉，那么如何找到该线程呢。我们看到**sleep指令**是无法查找的，只有通过performance_schema和sys系统来查询sys.schema_table_lock_waits表，就可以查找出阻塞process_id（Mysql需开启performance_schema=on）
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/9.png)

**等flush**

**等flush**过程中，会导致查询语句阻塞，通过如果对一个表做flush操作指令为
```java
// 只关闭表T 
flush tables t with read lock;

// 关闭所有表 
flush tables with read lock;
```
这两个指令本来是很快的，除非它们也被别的线程阻塞。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/10.png)

**等行锁**

```java
select * from t where id = 1 lock in share mode;
```
当我们执行这句查询时，id为1的记录会被加读锁，如果这时有一个事务在这行记录持有一个写锁，那么该查询语句就会阻塞，通过查询
```java
select * from t sys.innodb_lock_waits where locked_table=`'test'.'t'`
```
查询堵塞的源头，如图：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/11.png)

**一致性读**

处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/12.png)
对比上面两个sql，前者是普通的一致性读，后者是当前读，会用到读锁
如图：
处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/13.png)

sessionB在sessionA开启事务后update，如果更新100w次，那么就会生成100w个回滚日志。在第一个sql执行时就会依次执行uodolog，在100w次后，才将1结果返回

对于**查询条件字段**长度**大于**表结构**定义字段**长度的，mysql会截断该查询条件，然后进行查询，回表，返回


### 幻读
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/18.png)

如图:seesionA在T1,T3,T5阶段查询出来d=5返回的记录数都不一样,然而只有1,1,5这条记录,也就是session C **插入**的这条记录被称为**幻读**

我们所知,mysql数据innodb引擎**默认**事务隔离级别是**可重复读**,在可重复读隔离级别下,**普通查询**是**快照读**,是不会看到别的事务插入的数据的.因为上述select都是**for update** 都是**当前读**,拿到的最新的数据.因此幻读是在**新插入**数据和**当前读**情况下才出现

**幻读的影响**
**语义被破坏**,如上图在T1时刻,for update对5,5,5数据进行了加锁,而sessionB的操作破坏了sessionA想对所有d=5的记录加锁声明,破坏了**数据一致性**.如果我们在sessionA
T1时刻加一条语句,update t set d=100 where d=5;（update的加锁语义和select...for update是一致的),我们会发现sessionB和sessionC修改和增加的两条记录由于sessionA在T6时刻才提交，导致了这两条数据也被更改.
那么如何解决上述问题了

1. 在sesionA进行修改所有行的锁定，那么在sessionB进行操作的时候就会被阻塞，需要等待sessionA完成事务释放锁之后才能操作。但是我们会发现，这样并阻止不了sessionC的插入的记录的生成，也就是解决不了**幻读**的问题。
2. 要解决**幻读**的问题，那么我们就要引入间隙锁（Gap Lock，只能在可重复度级别下才可生效），间隙锁指的是两个记录之间的间隙，如果我们插入了5个记录那么就会产生6个间隙锁。在这些区间之间的数据受到该锁的保护的，解决了**幻读**的问题。
3. 两个间隙锁之间是**不会产生冲突**的，如果两个线程同时使用**同样的语句**产生两个相同的**间隙锁**，锁住了两个**相同区域**，而造成**死锁**，最终影响**并发**度。

**重要** 可重复读事务条件下的**加锁规则**

**原则**
1. 加锁基本单位是next-key lock，next=key lock是**前开后闭区间**
2. 查找过程中访问到的对象才会加锁

**优化**
1. 索引上的**等值查询**，给**唯一索引加锁**的时候，next-key lock退化为**行锁**
2. 索引上的**等值查询**，向**右遍历**时**最后一个值不满足**等值条件的时候，next-key lock退化为**间隙锁**

**bug**
1. **唯一索引**上的范围查询会访问到不满足条件的第一个值为止。


创建一个表和数据
```java
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB;

insert into t values(0,0,0),(5,5,5),
(10,10,10),(15,15,15),(20,20,20),(25,25,25);
```

案例1 **等值索引范围锁**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/19.png)

假如sessionA以事务执行update t set d=d+1 where id=7,根据加锁原则1，那么sessionA的加锁范围就是(5,10],虽然这是一个等值查询，而且是唯一索引，但是id=7并不存在，根据优化2，退化为间隙锁，最终加锁范围为(5,10); 

案例2  **非唯一索引等值锁**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/20.png) 

sessionA使用共享锁查询C=5的记录，根据原则1，(0,5]被锁，因为C是普通索引，所以继续往后查找到c=10，(5,10]被锁，根据优化2，退化为间隙锁（5,10），sessionB是使用的主键索引，针对覆盖索引，查询的对象只会在**覆盖索引上进行加锁**而不会影响主键索引，所以不会阻塞，而session C就会
 
这是覆盖索引的优化，如果我们有要求给**行加锁避免数据被更新**的话，就要在查询字段中加入**覆盖索引中不存在**的字段。

案例3 **主键索引范围锁**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/21.png) 

 sessionA先进行等值查询，本来是应该加锁(5,10]，但是由于id是唯一索引，根据优化1退化为行锁，继续向右遍历找到不满足条件的15，加锁(10,15]
 
案例4 **非唯一索引范围锁**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/22.png) 

 sessionA由于等值查询不是唯一索引，索引应该加锁(5,10]，根据bug1继续向右遍历找到不满足条件的15，加锁(10,15]
 
案例5 **唯一索引范围锁BUG**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/23.png) 

 根据原则1 范围查询，索引应该加锁(10,15]，sessionA是唯一索引，索引应该到15就停止了，但是根据bug1继续向右遍历找到不满足条件的20，原则2加锁(15,20]
 
案例6 **非唯一索引上存在等值的例子**：
```java
mysql> insert into t values(30,10,30);
```
这时数据库中有两条c=10的记录
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/24.png)

有如下事务
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/25.png)

根据原则1，锁住(5,10]，非唯一索引，优化2，继续向右遍历锁住(10,15）

案例7 **limit语句加锁**：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/26.png)

我们在思考为什么会产生死锁呢？sessionA加锁(5,10]，(10,15），sessionB不是应该申请不到锁吗？
实际是这样的先加锁(5,10），然后加锁10，被锁住


### mysql临时提高性能的方案
 
**短连接风暴**
 
短连接模式中,每一次连接只会执行很少的sql语句,当数据库处理慢的时候,短时间并发上去之后,连接数上去后,之后的请求都会被提示"too many connections"的错误.但是又不能无限的增加max_connections的上限(相应的会增加系统负载,资源消耗,权限验证等逻辑).

1. 在max_connections中有很多连接都是空闲的,可以通过设置wait_timout被动**剔除空闲连接**
2. 主动kill空闲连接,
    - 通过show processlist,查看处于**sleep**中的线程
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/14.png)
如图:有两个空闲连接,这两个空闲连接,有的是**事务中的空闲连接**,有的是**事务外空闲连接**,我们优先kill掉**事务外空闲连接**.

    - 查询事务具体状态
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/15.png)
如图:通过上述sql查询处于事务中的连接为4,然后使用kill connection id执行kill.

被kill掉的客户端在下一次请求时会收到**Lost connection to MySQL server during query** 报错，从数据库主动断开连接是有损的，如果应用端收到错误后不重新连接，而是直接用这个句柄继续查询，会导致误认为mysql没有恢复
3. 减少连接过程消耗
    - 跳过权限验证阶段，重启数据库，并使用-skip-grant-tables参数启动（在8.0版本默认启用该功能后，--skip-networking参数也会打开，只允许本地客户端连接）
4. 慢查询解决
    - 索引没建好，在5.6以后版本使用Online DDL创建紧急索引。1先在备库执行，set sql_log_bin=off，不写binlog，执行alter table加索引；2然后执行主备切换；3然后再在切过来的主库（现在是备库）执行一遍第一步操作
    - 语句没写好，5.7提供了query_rewritegongn ,可以把输入一种语句变成另外一种模式
    - mysql选错索引，这种情况向下，通过前面所讲的使用force index
5. QPS突增
    - 一种是全新业务的bug导致，如果DB运维规范，可以直接从白名单下掉该功能
    - 如果新功能时单独数据库用户，可以用管理员账号把用户删掉，断开现有连接。
    - 如果新功能跟主题功能部署一起，那么只能通过处理语句限制，查询重写把sql改为select 1返回

	
#### binlog写入机制
事务执行过程中,先把日志写到binlog cache,事务提交的时候,再把binlog cache写到binlog中. 系统给binlog cache分配了内存,每个线程一个, binlog_cache_size用于控制单个线程内所占内存大小.每个线程有独立的binlog cache,但是共用同一份binlog文件,如果超过了这个参数大小,就要暂存到磁盘.

如图:
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/27.png) 

图中的write,指的是把日志写入到page cache;图中的fsync,是将数据持久化磁盘的操作.

write和fsync是由sync_binlog控制的.(比较常见的设置为100~1000中某个值)

- sync_binlog=0,每次提交事务只write,不fsync;
- sync_binlog=1,每次提交事务只fsync;
- sync_binlog=N(N>1),每次提交事务都write,但累积N个事务后才fsync.

#### redolog写入机制
rodolog是先写到redo log buffer里的,不是每次生成后就持久化到磁盘的,也不是非得事务提交的时候,redo log buffer中日志才持久化到磁盘.

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/28.png) 
上图对应redolog的三种状态

- 存在redo log buffer中,物理上处于mysql进程内存中,红色部分;
- 写到磁盘(write),但是没有持久化(fsync),物理上是在文件系统中page cache中,也就是黄色部分;
- 持久化到磁盘,对应hard disk,也就是绿色部分.

速度方面红色和黄色部分速度相当,绿色部分要慢一些

同样redolog也有写入策略控制参数,innodb_flush_log_at_trx_commit有三种取值

- 设置为0的时候,每次事务提交都只是把redo log 留在redo log buffer中
- 设置为1的时候,每次事务提交时都将redo log直接持久化到磁盘
- 设置为2的时候,每次事务提交时都只把redo log写到page cache

注意:事务执行中间过程redo log也会写在redo log buffer中.


有一个后台线程,每隔一秒,就会把redo log buffer 中日志,调用write写到文件系统的page cache.然后fsync持久化到磁盘.除此以外,还有两种场景redo log 写入到磁盘中.1.redo log buffer占用空间即将达到innodb_log_buffer_size一半时,会主动进行写盘(只是到达page cache中).2
另一种是,并行的事务提交的时候,顺带把redo log buffer持久化到磁盘.如果把innodb_flush_log_at_trx_commit参数设置成1,每秒一次的轮询刷盘,再加上崩溃恢复机制,innodb认为redo log在commit时候就不需要fsync了,只会write到文件系统中的page cache.

如果我们采用双1配置,也就是binglog和redolog刷盘策略都配置1,那么每秒的磁盘TPS就等于4w,如果磁盘能力也就2w左右,那么怎么保证TPS呢?

**组提交**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/29.png) 

如图有三个事务,分别先后到达mysql进程内存中,t1先到达,被选举为这组的leader,等T1写盘的时候,LSN变成了160,那么小于160的redolog都会被顺带持久化到磁盘.也就是t2和t3可以直接返回了.一次组提交里面,组员越多,节约的IOPS效果越好.

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/30.png) 

如图：我们前面所说两阶段提交，实际上是这样的，这样binlog和redolog都可以进行组提交。

由于第三步非常快，索引binlog的组提交效果并不明显，可以使用**binlog_group_commit_sync_delay**设置多少微秒后才调用fsync，**binlog_group_commit_sync_no_delay_count**等待多少事务数量才调用fsync。这两种方式可能增加语句的响应时间，但没有丢失数据的风险

前面所讲的WAL机制是**减少磁盘写**，这个机制得益于两个方面：1redo log和binlog都是顺序写 2组提交机制，可以大幅降低磁盘IOPS消耗

总结：磁盘性能出现在IO方面的瓶颈解决方案
- 设置**binlog_group_commit_sync_delay**，**binlog_group_commit_sync_no_delay_count**参数，减少binlog写盘次数，安全的
- 将sync_binlog设置为大于1的值,有风险，主机掉电会丢失binlog日志。
- 将innodb_flush_log_at_trx_commit设置为2，有风险，也是主机掉电会丢失binlog日志。

crash-safe保证
1. 如果客户端收到事务成功消息，事务就一定持久化了
2. 如果客户端收到事务失败的消息，事务就一定失败了
3. 如果客户端收到执行异常的消息，应用重连后通过查询当前状态继续后续逻辑，此时数据库只需保证内部一致就可以了。

#### 主备同步
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/31.png) 

上图是主备同步的流程，备库B跟主库A之间维持了一个长连接

- 在备库B上通过change master命令，设置主库A的IP，端口，用户名，密码以及从哪个位置开始请求binlog，这个位置包含文件名和日志偏移量
- 在备库B执行start slave命令，这时会启动两个线程，如图，其中IO_thread负责与主库建立连接
- 主库A校验完用户名，密码，按照备库B传过来的位置，从本地读取binlog，发给B.
- 备库拿到binlog后，写到本地文件，称为中转日志
- sql_thread读取中转日志，解析出日志里命令，并执行

后期多线程复制方案的引入，sql_thread演化为多个线程

#### binlog的三种格式对比
statement 可能会导致binlog**主备不一致**，（比如delete语句加上limit时）
row 很**占空间**，如果用delete语句删除10w行数据，用statement就只是一个sql语句被记录到binlog中，而用row，就要把10w条记录写到binlog中，而且还耗费IO资源，但是现在越来越多的场景使用row格式，原因是**恢复数据**的好处
mixed：综合上面两种方式，**判断这条sql是否可能引起主备不一致**，如果可能，就用row格式，否则用statement格式。

binlog重放应该使用mysqlbinglog工具解析出来，然后把解析结果整个发给mysql执行，而不是把里面的statment语句拷贝出来执行

主备切换常用结构，双M结构和M-S结构，区别在于节点A和B之间总是互为主备关系，那么在主备切换的时候会不会导致binlog之间重复循环执行呢？会有这种情况，解决方案：

- 规定两个库的serverid必须不同，如果相同，则它们，不能互为主备关系
- 一个备库接到binlog并重放过程中，生成与原binlog的server id相同的新的binlog
- 每个库收到从自己主库发来日志后，先判断serverid，如果相同，表示日志是自己生成，直接进行丢弃

正常情况下，主库生成的binlog都能传到备库并正确的执行，备库就能达到跟主库一致的状态。但是mysql要实现高可用，一致性是远远不够的。

主备切换可能是在正常运维下，也可能是在被动操作，如主库所在机器掉电。

### 主备延迟
1. 主库A执行完一个事务，写入binlog，把这个时刻记为T1
2. 之后传给备库B，我们把备库B接收完这个binlog时刻记为T2
3. 备库B执行完这个事务，记为T3

主备延迟=T3-T1，用seconds_behind_master参数表示，show slave status可以得到主备延迟

常情况下T2-T1之间的值很小的。也就是主备延迟主要来源是**备库接收完binlog和执行完这个事务之间的时间差**

**原因**
1. 备库所用机器较主库差一点，但是在更新请求对IOPS的压力是一样的，主备是无差别的。（现在很少这种部署方式）
2. 备库压力大。主库主要提供写能力，备库提供读能力，当备库读取压力大时，影响了同步速度，造成了主备延迟。

    - **解决方案**
    1. 一主多从。除了备库外，可以多接几个从库，让从库分担读的压力。（从库还适合做备份）
    2. 通过binlog输出到外部系统，比如hadoop，让外部系统提供统计类查询能力
3. 大事务，主库上必须等事务执行完成才会写入binlog，再传给备库，如果一个语句执行10分钟，那么从库延迟至少会10分钟。（一次性用delete语句删除太多数据；大表DDL）

**解决方案**
1. 一主多从。除了备库外，可以多接几个从库，让从库分担读的压力。（从库还适合做备份）
2. 通过binlog输出到外部系统，比如hadoop，让外部系统提供统计类查询能力


### 主备切换策略

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/32.png) 

**可靠性优先策略**
1. 判断备库B现在的seconds_behind_master，如果小于某值，进行下一步，否则重试
2. 把主库A改成只读状态
3. 判断备库B的seconds_behind_master的值，直到这个值变为0为止（第一步判断sbm的值很重要，决定了系统不可用时间）
4. 把备库B改成可读写状态
5. 把业务请求切到备库B

**可用性优先策略**
直接强行把步骤4,5调整到最开始执行，系统几乎没有不可用时间，但是会出现数据不一致的情况。

**如果我们设置binlog_format=mixed**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/33.png) 
，在主库完成插入4，4数据时，正要发送binlog到备库时，主备进行了切换，那么备库继续执行本该主库执行的5,5数据确成了4,5。

**如果我们使用binlog_format=row**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/34.png) 因为row格式在记录binlog时，会记录新插入的行的所有字段值，所以最后只会有一行不一致，备库5,4和主库5,5都不会被对方执行，主备同步线程会duplicate key error报错


**可靠性策略的异常切换**
如果主库掉电，那么在这段时间内，中转日志还没应用完成，如果直接发起主备切换，客户端查询不到之前完成的事务，会认为“数据丢失 ”

#### 主备并行复制能力
在之前说到的主库完成写入事务和备库sql_thread线程执行中转日志，真实情况下sql_thread的更新数据速度是小于主库的写入的（主库影响并发度主要是锁，5.6之前mysql只支持单线程复制，在主库并发高，TPS高时就会出现严重主备延迟问题）。

所有的多线程复制机制都是把一个线程拆分成多个线程。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/35.png) 

图中coordinator就是原来的sql_thread，只负责读取中转日志和分发事务。worker线程进行日志的更新。work线程的个数，就是由参数slave_parallel_workers决定。
因为是并行更新，不管是一个事务拆分，还是多个事务分发，都有可能最终造成数据的不一致。那么不管coordinnator必须满足：
- 不能造成更新覆盖。更新同一行两个事物必须分发到同一个worker中
- 同一个事务不能被拆开，必须放到同一个worker中

#### 5.5之前并行复制策略

**按表分发策略**
思路：如果两个事务更新不同的表，它们就可以并行，可以按表分发，两个worker不会更新同一行，如果有跨表的事务，还是要把两张表放在一起考虑的。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/36.png) 

每个work对应一个hash表，用于保存worker执行队列里的事务涉及的表。key是“库名.表名”，value是一个数字，表示队列中有多少个事务修改。

如图worker1和worker2分别修改t1和t3，如果这时有一个新事务T，涉及T1,T3,因为w1和w2都涉及T中的修改表，那么coordinator就进入等待。如果table2里涉及表T3里面事务先完成，那么只有w1跟T有冲突了，那么T就分配到w1中，而coordinator继续下一个中转日志。

**规则**
1. 如果跟所有worker都不冲突，coordinator会把事务分配给最闲的worker
2. 如果多于一个worker冲突，coordinator进入等待，直到冲突只剩下一个，并分配给该线程

- 缺点是如果碰到热点表的场景，那么就变成单线程复制了

**按行分发策略**
思路：如果两个事务没有更新相同的行，它们在备库上可以并行执行，这个模式要求binlog格式必须是row。

按行分发的事务与worker是否冲突，判断变成了从**表**变成**行**， key变成了“库名+表名+唯一键的值”。

仅仅是有唯一键是不够的，如果还有唯一索引，而更新两行数据的唯一索引值可能会报唯一键重入，key应为“库名+表名+索引a的名字+索引a的值”。


- 缺点是按行并行策略，因为计算hash需要消耗更多的计算资源

上面两种策略有约束：1要能从binlog中解析出表名，主键，唯一索引，2表必须有主见3不能有外键
#### 5.6并行复制策略
按库并行

- 优势：构造hash值会很快，只需要库名。2不要求由binlog格式

#### MariaDB的并行复制策略
运用redolog组提交的优化得到

1. 能够在同一组提交的事务，一定不会修改同一行
2. 主库上可以并行执行的事务，备库上也一定可以并行执行

**做法**
1. 在一组里面一起提交的事务，有一个相同的commit_id,下一組就是commit_id+1
2. commit_id直接写入binlog里面
3. 传到备库时，相同commit_id分发到多个worker执行
4. 一组执行完，coordinator再取下一批

- 缺点：备库执行，要等第一组事务完全执行完，第二组才能开始执行，系统吞吐量不够；容易被大事务拖后腿

#### 5.7的并行复制策略

有两种模式 1 按照库并行策略 2 类似于MariaDB的并行复制策略

如果事务脱离由于锁冲突而处于锁等待状态时，就可以并行。 binlog组提交时拉长组提交时间可以减少binlog写盘次数，同时增加备库复制并行度

1. 同时处于prepare状态的事务，备库可以并行执行
2. 处于prepare状态事务和处于commit状态事务，备库执行可以并行

#### 5.7.22的并行复制策略

WRITESET并行复制策略
binlog-transaction-dependency-tracking
1. commit_order：同时根据prepare和commit判断是否可以并行策略
2. WRITESET：表示对事务涉及更新的每一行，计算每一行hash，组成集合WRITESET，如果两个事务没有操作相同的行，就可以并行。
3. WRITESET_SESSION：是在WRITESET基础上多了一个约束，即在主库上同一个线程先后执行两个事务，备库执行时，要保证相同先后顺序。

WRITESET比前面5.6之前实现还是有改良，
1. WRITESET是在主库生成后直接写入binlog里面的，这样在备库执行时，不需要解析binlog内容。节省了计算量
2. 不需要吧整个事务binlog都扫一遍才决定分发到哪个worker，更省内存
3. 备库分发策略不依赖于binlog格式

 
 
#### 一主多从架构
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/37.png) 

**基于位点的主备切换**
当我们要把节点B设置成节点A'的从库的时候，需要执行一条change master命令
```java
CHANGE MASTER TO 
MASTER_HOST=$host_name // 主库A'的ip
MASTER_PORT=$port // 主库A'的端口
MASTER_USER=$user_name // 主库A'的用户名
MASTER_PASSWORD=$password // 主库A'的密码
MASTER_LOG_FILE=$master_log_name // 主库同步文件
MASTER_LOG_POS=$master_log_pos  // 主库同步位点
```

取同步位点的方法：
1. 等待新主库A'把中转日志全部同步完成
2. 在A'上执行show master status命令，得到当前A'上最新的file和position
3. 取原主库A故障的时刻T
4. 用mysqlbinlog工具解析A'的file，得到T时刻的位点

```java
mysqlbinlog File --stop-datetime=T --start-datetime=T
```
T时刻写入新的binlog的位置以end_log_pos参数给出。但是这个参数并不准确（如果T时刻，主库A以及执行完成一个insert语句插入一行数据R，并将binlog传给了A'和B，然后A就掉电了，那么此时1从库B上因为同步了binlog，R这一行已经存在 2在新主库A'上,R这一行也已经存在，在123位置之后，3在B上执行change master命令，指向A'的123位置，那么插入R这一行又会同步到B）

**解决方案**
1. 主动跳过一个事务
```java
set global sql_slave_skip_counter=1;
start slave;
```
切换过程中，可能不止重复执行一个事务，B接到A'时，会进行持续观察，每次碰到错误就停下来，跳过。

2. 设置slave_skip_errors参数，直接跳过指定的错误(1062错误：插入数据唯一键冲突；1032错误：删除数据时找不到行)

**GTID**
通过sql_slave_skip_counter跳过事务和通过slave_skip_errors忽略错误可以重新建立库B和新主库A'的主备关系。但是这两种操作很复杂，而且容易出错。

GTID全称Global Transaction Identifier，也就是全局事务ID。表示一个事务提交的时候生成的，是这个事务的唯一标识。
```java
GTID=server_uuid:gno // GTID格式
```

- server_uuid是一个实例第一次启动时自动生成的，全局唯一。
- gno是一个整数，初始值时1，每次提交事务时分配这个给事务，并+1。

启动GTID模式，加上gtid_mode=on 和 enforce_gtid_consistency=on参数

在GTID模式下，每个事务都会跟一个GTID一一对应，这个GTID有两种生成方式，而使用哪种方式取决于session变量gtid_next的值

**基于GTID的主备切换**
备库B设置为新主库A'的从库
```java
CHANGE MASTER TO 
MASTER_HOST=$host_name 
MASTER_PORT=$port 
MASTER_USER=$user_name 
MASTER_PASSWORD=$password 
master_auto_position=1  // 表示主备关系使用GTID协议
```
设A'的GTID集合记为set_a，B的GTID集合记为set_b。

我们在B库上执行start slave命令

1. 实例B指定A'，基于主备协议建立连接
2. 实例B把set_b发给主库A'
3. 实例A'算出set_a与set_b的差集，也就是所有存在于set_a，但不存在于set_b的GTID集合，判断A'本地是否包含这个差集所需所有binlog事务
    - 如果不包含，表示A'已经把实例B需要的binlog给删除了，直接返回错误。
    - 如果确认全表包含，A'从自己的binlog文件里面，找出第一个不在set_b的事务，发给B
4. 之后从这个事务开始，往后读文件，按顺序取binlog发给B执行。

相比于基于位点的主备协议，前者由备库决定，备库指定哪个位点，主库就发哪个位点，不做日志完整性判断；后者认为只要建立主备关系，主备就必须保证发给备库的日志时完整的。

**GTID和在线DDL**
在双M结构下，备库执行DDL语句也会传给主库，为了避免传回后对主库造成影响，要通过set sql_log_bin=off关掉binlog


### 主库延迟（过期读）的处理
**强制走主库方案**
将查询请求做分类
- 对于必须要拿到最新结果的请求，强制将其发送到主库上。
- 对于可以读到旧数据的请求，才将其发到从库上。

**sleep方案**
主库更新后，读从库之前先sleep一下。类似于一条select sleep(1)命令。 

**判断主备无延迟方案**

- 通过每次从库查询请求前，判断sbm是否等于0。如不等于0，需等待变为0才执行查询请求
- 对比位点确保主备无延迟，通过master_log_file和read_master_log_pos，表示读到的主库最新位点；Relay_Master_Log_File 和 Exec_Master_Log_Pos，表示备库执行最新位点。如这两组位点一致，表示接收到的日志已经完全同步
- 对比GTID集合确保主备无延迟
    - auto_position=1，表示对主备协议使用GTID协议
    - Retrieved_Gtid_Set，是备库收到的所有日志的GTID集合
    - Executed_Gtid_Set，是备库所有已经执行完成的GTID集合
    两个集合进行比较，如一致，表示同步已完成

是不是有了这些方案，过期读就解决了呢？严格意义上没有，因为在判断是否同步完成时，主备之间可能还是会有刚好产生的日志而且未同步。

**semi-sync replication**
设计思路
- 事务提交的时候，主库把binlog发给从库
- 从库收到binlog以后，发回给主库一个ack，表示收到
- 主库收到ack，给客户端返回“事务完成确认”

- 缺点:一主多从的时候，在某些从库执行查询请求会存在过期读的现象 2在持续延迟的情况下，可能出现过度等待的问题

**等主库位点方案**
```java
select master_pos_wait(file, pos[, timeout]);
```
指令逻辑如下：

1. 从库执行
2. 参数file和pos值的是主库上文件名和位置
3. timeout可选，设置为正整数N表示这个函数最多等待N秒

指令返回结果是一个正整数M，表示从命理开始执行，到应用完file和pos表示的binlog位置，执行了多少事务。附带（执行期间，备库同步线程发生异常，则返回NULL 2等待超过N秒，就返回-1 3开始执行时，就发现已经执行过这个位置，就返回0）

1. trx1 事务更新完成后，马上执行 show master status 得到当前主库执行到的 File和Position；

2. 选定一个从库执行查询语句在从库上执行 select master_pos_wait(File, Position, 1)；
3. 如果返回值是>=0的正整数，则在这个从库执行查询语句；
4. 否则，到主库执行查询语句。

**等GTID方案**
```JAVA
 select wait_for_executed_gtid_set(gtid_set, 1);
```
指令逻辑

- 等待，直到这个库执行的事务中包含传入的 gtid_set，返回 0； 
- 超时返回 1。
 
1. trx1 事务更新完成后，从返回包直接获取这个事务的 GTID，记为 gtid1； 
2. 选定一个从库执行查询语句；
3. 在从库上执行selectwait_for_executed_gtid_set(gtid1, 1)；
4. 如果返回值是0，则在这个从库执行查询语句；
5. 否则，到主库执行查询语句。
 
### 如何判断数据库出问题

**select 1**
select 1 只能说明库还在，不能说明主库没有问题

我们设置innodb_thread_concurrency（通常设置为64~128之间的值），控制InnoDB的并发线程上限。
如图：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/38.png) 
可以看出使用select 1是检测不出实例是否正常的

**并发连接，并发查询**
并发连接高并不会影响CPU，就多占用内存，并发查询高才是CPU杀手

热点数据查询，线程进入锁等待以后，并发线程计数会减1

**检测innodb并发线程过多导致系统不可用情况**
```java
mysql> select * from mysql.health_check; 
```
但是如果空间满了后，这个检测语句哟不好用了，更新事务写binlog，磁盘满了后，commit语句会堵住。但是系统可以正常读取数据

解决方案：使用更新语句
```java
mysql> update mysql.health_check set t_modified=now();
```

主备都应该进行检测，如果主备是双M结构，那么主备都用相同更新命令会导致主备同步停止。

解决方案：在mysql.health_check增加多行数据，并用A,B的servier_id做主键

缺点：判定慢

更新语句，如果失败或者超时，就可以发起主备切换了，为什么还会有判定慢的问题呢？

首先，所有检测逻辑都需要一个超时时间N，执行一条update，超过N秒不返回，认为系统不可用（日志盘IO利用率100%场景）

**内部统计**
mysql5.6以后提供的performance_schema库，在file_summary_by_event_name表里统计了每次IO请求的时间。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/39.png) 

因为我们每一次操作数据库，performance_schema都需要额外地统计这些信息，所以我们打开这个统计功能是有性能损耗的。如果打开所有的 performance_schema 项，性能大概会下降 10% 左右

打开特定的统计数据
 ```java
 mysql> update setup_instruments set ENABLED='YES', Timed='YES' where name like '%wait/io/file/innodb/innodb_log_file%';
 ```
 
检测逻辑
```java
 mysql> select event_name,MAX_TIMER_WAIT  FROM performance_schema.file_summary_by_event_name where event_name in ('wait/io/file/innodb/innodb_log_file','wait/io/file/sql/binlog') and MAX_TIMER_WAIT>200*1000000000;
```

清空统计信息
```java
mysql> truncate table performance_schema.file_summary_by_event_name;
```

### 误删数据处理

#### **误删行**
使用delete语句误删行，可用flashback工具通过闪回恢复

原理：通过修改binlog内容，拿回数据库重放，前提是确保binlog_format=row和binlog_row_image=FULL

单个事务处理:
1. 对于insert语句，对应binlog_event类型是write_rows_event，把它改成delete_rows_event
2. 同理，对于delete语句，将delete_rows_event改成write_rows_event
3. 对于update_rows，binglog里面对调修改前和修改后的值即可。

多个事务处理：
用flashback解析后，要将事务的顺序调过来在执行

**恢复数据比较安全的做法是恢复出一个备份，在临时库上执行操作，然后在确认过的备份，恢复回主库**

**事先预防**
1. 把sql_safe_updates参数设置为on。如果delete或update语句中where条件，没有包含索引字段，语句执行就会报错
2. 代码上线前，需审计

#### **误删库/表**
要求需要平时有全量备份，加增量日志的机制。

1. 取最近一次全量备份，
2. 用备份恢复出一个临时库
3. 从日志备份里，取出凌晨0点后的日志
4. 把这些日志，除了误删外，全部应用到临时库

**注意**

- 为了加速数据恢复，如果临时库有多个数据库，可以使用mysqlbinlog命令加上一个-datebase参数，指定误删表所在的库
- 第3步应用日志时，需要跳过误删除的语句的binlog
    - 如果原实例没有使用GTID模式，只能包含误删语句的binlog文件时，先使用-stop-postion参数执行误操作之前日志，然后使用-start-position继续执行误操作之后的日志。
    - 如果使用了GTID模式，假设误操作命令GTID是gtid1，那么只需执行set_gtid_next=gtid1;begin;commit;先把这个GTID加到临时实例的GTID集合，之后按顺序执行binlog，就会自动跳过误操作语句

**缺点**：
mysqlbinlog并不能指定解析一个表的日志；用mysqlbinlog解析出日志应用，应用日志过程只能是单线程

**加速方法**
在start_slave之前，先通过执行change replication filter replicate_do_table=(tbl_name)命令，就可以让临时库只同步误操作的表。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/40.png) 

如果时间太久线上备库已经删除临时实例需要的binnlog的话，我们可以从binlog备份系统中找到需要的binlog，再放回备库中

例:当前临时实例需要binlog从master.05开始的，备库最小binlog时master.07。则进行如下操作：

1. 从备份系统下载master.05和master.06两个文件，放到备库日志目录下
2. 打开日志目录下master.index文件，文件开头加入两行，./master.05,./master.06
3. 重启备库，让备库重新识别这两个日志文件
4. 现在备库上有了临时库所需所有binlog，建立主备关系，可以正常同步了

同时经常进行数据恢复的演练也是很重要

#### **延迟复制备库**

如果一个库的备份特别大，误操作时间距离上一个全量备份时间较长，而且我们有非常核心的业务，不允许太长的恢复时间，考虑搭建延迟复制的备库

通过CHANGE MASTER TO MASTER_DELAY = N命令，可以指定这个备库持续保持跟主库有N秒的延迟，如果在N秒内发现了误删的数据，那么先到备库执行stop slave，再通过之前的方法，跳过误操作命令，恢复出需要的数据

#### **预防误删库/表的方法**
- 账号分离
    - 业务开发同学只给DML权限，不给truncate/drop权限
    - 平常只是用只读账号，必要时使用更新权限账号
- 规范操作
    - 删除数据表前，先对表做改名操作，观察一段时间，确保业务无影响后再删除这张表
    - 改表名，要求给表名加固定后缀，通过管理系统执行删除表，删除表时，只能删除固定后缀的表

#### **rm删除数据**
手动狗头，只要不是所有集群都删了，单个节点数据的恢复，然后再接入整个集群


### 为什么有kill不掉的语句

**kill query thread_id_B**
**线程没有执行到判断线程状态的逻辑情况**
kill掉查询语句的流程：
1. 把session B 运行状态改成 THD::KILL_QUERY（将变量killed赋值为THD::KILL_QUERY）
2. 给session B 执行线程发一个信号（发通知是为了让sessionB退出等待，来处理这个THD::KILL_QUERY状态）

从整个流程可以得到三层信息:
- 一个语句执行过程中有多处埋点，这些埋点的地方判断线程状态，如果线程状态是THD::KILL_QUERY，才开始进入语句终止逻辑
- 如果处于等待状态，必须是一个可以被唤醒的等待，否则不会执行到埋点处
- 语句从开始进入终止逻辑，到终止逻辑完全完成，是有一个过程的

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/41.png) 

如图先设置global innodb_thread_concurrency=2。我们发现
- session C执行时候被堵住了
- session D执行kill query C命令却没什么效果
- 直到sessionE执行了kill connection命令，才断开sessionC的连接
- 我们执行show processlist，看见如下图

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/42.png) 

sessionD执行kill后，sessionC还是没有退出。因为12号线程每10ms判断一下是否可以进入innodb执行，如果不行，调用nanosleep进入sleep状态

seesionE执行kill connection命令时，
1. 把12号线程状态设置为 KILL_CONNECTION
2. 关掉12号线程的网络连接，因为这个操作，你会看到，sessionC收到了断开连接的提示
3. 执行show processlist，会看到command显式killed

即使客户端退出，这个线程仍处于等待中，只有等到满足进入innodb条件后，sessionC继续执行，才判断线程状态变成了kill_query和kill_connection,再进入终止逻辑阶段

**终止逻辑耗时较长**
从show processlist结果上看也是command=killed。常出现于以下场景

- 超大事务执行期间被kill。回滚操作需要对事务执行期间生成新数据版本做回收操作，耗时长
- 大查询回滚。如果查询过程生成了较大临时文件，加上此时文件系统压力大，删除临时文件需要等待IO资源，导致耗时长
- DDL命令执行到最后，如果被kill，需要删除中间过程临时文件，可能受IO资源影响耗时久

**为什么客户端连接有时很慢**
当使用默认参数连接时，mysql客户端提供一个本地库名和表名补全功能 1.执行show databases 2切到db1库，执行showtables3把这两个命令结果用于构建一个本地哈希表

这种慢是客户端慢而不是服务端慢，可以使用连接命令加上 -A，关闭自动补全功能

加-quick也可以跳过补全阶段，但是这个参数可能会降低服务端性能

mysql发送请求后，接收服务端返回结果方式：
1. 本地先把结果缓存起来
2. 不缓存，读一个处理一个

客户端默认使用第一种，加上-quick就会使用第二种不缓存方式

**-quick效果**
- 跳过表名自动补全
- mysql_store_result需申请本地内存缓存查询结果，缓存过大会影响本地机器性能
- 不会吧执行命令记录到本地命令历史文件

### 全表扫描对server的影响
假如对200G的innodb表db1.t，执行全表扫描，
```java
 mysql -h$host -P$port -u$user -p$pwd -e "select * from db1.t" > $target_file
```

如图，产生一个结果集，服务端取数据，发数据流程：
1. 获取一行，写到net_buffer中，由net_buffer_length定义，默认16k
2. 重复获取行，直到net_buffer写满，调用网络接口发出去
3. 发送成功，清空net_buffer，继续下一行，写入net_buffer
4. 如果返回EAGIAN或WSAEWOULDBLOCK，表示本地网络栈写满，进入等待。直到网络栈重新科协，再继续发送

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/43.png) 

从这个流程，你可以看到：
1. 一个查询在发送过程中，最多占用内存就是net——buffer_length这么大
2. socket_send_buffer也不能达到200G

结论：**MySQL 是边读边发的**
如果客户端不去读socket_send_buffer中内容，然后在服务端 show processlist
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/44.png) 
State 的值一直处于“Sending to client”，就表示服务器端的网络栈写满了。
 
 使用–quick 参数，会使用 mysql_use_result 方法，读一行处理一行，如果业务的逻辑比较复杂
服务端就会很慢

**如果一个查询的返回结果不会很多，建议使用 mysql_store_result接口，直接把查询结果保存到本地内存。
 
 
一个查询语句的状态变化：
- MySQL 查询语句进入执行阶段后，首先把状态设置成Sending data
- 然后，发送执行结果的列相关的信息（meta data) 给客户端；
- 再继续执行语句的流程；
- 执行完成后，把状态设置成空字符串。


**一个线程处于“等待客户端接收结果”的状态，才会显示"Send to client"；而如果显示成“Sending data”，它的意思只是“正在执行**

### 全表扫描对 InnoDB 的影响
内存的数据页是在 Buffer Pool (BP) 中管理的，在 WAL 里 Buffer Pool 起到了加速更新的作用，查询要读数据页，如果内存数据页的结果是最新的，直接从内存拿结果
，Buffer Pool 对查询的加速效果，依赖于一个重要的指标，**内存命中率**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/45.png) 

innodb_buffer_pool_size建议设置成可用物理内存的 60%~80% 

Buffer Pool 满了，而又要从磁盘读入一个数据页
，InnoDB 内存管理用的是最近最少使用 (Least Recently Used, LRU) 算法。

假设按照这个算法，我们要扫描一个 200G 的表，会把当前的 Buffer Pool 里的数据全部淘汰掉，存入扫描过程中访问到的数据页的内容

改进后的LRU：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/46.png) 

按照 5:3 的比例把整个 LRU 链表分成了 young 区域和 old 区域

操作逻辑：
- 扫描过程中，需要新插入的数据页，都被放到 old 区域 ;
- 一个数据页里面有多条记录，这个数据页会被多次访问到，但由于是顺序扫描，这个数据页第一次被访问和最后一次被访问的时间间隔不会超过 1 秒，因此还是会被保留在 old 区域
- 再继续扫描后续的数据，之前的这个数据页之后也不会再被访问到，于是始终没有机会移到链表头部（也就是 young 区域），很快就会被淘汰出去

### join的使用

#### 驱动表和被驱动表的选择
驱动表是走全表扫描，而被驱动表是走树搜索。

- 使用 join 语句，性能比强行拆成多个单表执行SQL 语句的性能要好；
- 如果使用 join 语句的话，需要让小表做驱动表。

**结论**：小表作为驱动表，**前提**：可以使用被驱动表的索引

#### 用不上索引的情况
Simple Nested-Loop Join算法 ：扫描次数=驱动表行数*被驱动表行数，算法太笨重

Block Nested-Loop Join算法
1. 把表 t1 的数据读入线程内存 join_buffer 中，由于我们这个语句中写的是 select *，因此是把整个表 t1 放入了内存；
2. 扫描表 t2，把表 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回。

从时间复杂度上来说，这两个算法是一样的。但是Block Nested-Loop Join 算法的判断是内存操作，速度上会快很多，性能也更好。

判断要不要使用 join 语句时，就是看 explain 结果里面，Extra 字段里面有没有出现Block Nested Loop”字样。

**结论**：总是应该使用小表做驱动表

**什么是小表**
- 小表包含用上条件过滤的表
- 数据量小的是小表

**在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表。**

### join的优化
主键索引是一棵 B+ 树，在这棵树上，每次只能根据一个主键id 查到一行数据。因此，回表肯定是一行行搜索主键索引的，随着 a 的值递增顺序查询的话，id 的值就变成随机的，就会出现随机访问性能相对较差。虽然“按行查”这个机制不能改，但是调整查询的顺序，还是能够加速的。

**Multi-Range Read**

- 根据索引 a，定位到满足条件的记录，将 id 值放入 read_rnd_buffer(read_rnd_buffer_size控制)中 ;
- 将 read_rnd_buffer 中的 id 进行递增排序。
- 排序后的 id 数组，依次到主键 id 索引中查记录，并作为结果返回。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/48.png) 
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/49.png) 
Extra 字段多了 Using MRR，表示的是用上了 MRR 优化
在 read_rnd_buffer 中按照 id 做了排序，得到的结果集也是按照主键 id 递增顺序

**Batched Key Access**
NLJ 算法执行的逻辑是：从驱动表 t1，一行行地取出a 的值，再到被驱动表 t2 去做 join。也就是说，对于表 t2 来说，每次都是匹配一个值。这时，MRR 的优势就用不上了。我们需要构造一个临时内存，join_buffer，数据取出来一部分

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/50.png) 

启用BKA算法：
```java
set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
```

#### 临时表

**特征**
1. 建表语法是 create temporary table …
2. 一个临时表只能被创建它的 session 访问，对其他线程不可见
3. 临时表可以与普通表同名。
4. session A 内有同名的临时表和普通表的时候，show create 语句，以及增删改查语句访问的是临时表
5. show tables 命令不显示临时表
6. session结束时，会自动删除临时表

**应用**
由于不用担心线程之间的重名冲突，临时表经常会
复杂查询的优化过程中

- 分库分表系统的跨库查询(一般分库分表的场景，就是要把一个逻辑上的大表分散到不同的数据库实例上。) 比如。将一个大表 ht，按照字段 f，拆分成 1024 个分表，然后分布到 32 个数据库实例上.
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/51.png) 
    - 通常，分库分表系统都有一个中间层proxy。分区key的选择以“减少跨库和跨表查询”为依据，如果大部分语句包含f的等值条件，那么就使用f作为分区键，proxy解析完sql就知道到哪个分表进行查询
    - 如果这个表上还有其他索引k，而查询语句并没有用到分区字段，那么有两种办法
        - 在 proxy 层的进程代码中实现排序。(优点：处理速度快，拿到分库的数据以后，直接在内存中参与计算;缺点：需要的开发工作量比较大；对 proxy 端的压力比较大，尤其是很容易出现内存不够用和CPU 瓶颈的问题
        - 把各个分库拿到的数据，汇总到一个 MySQL 实例的一个表中,然后在这个汇总实例上做逻辑操作


**临时表重名**
```java
create temporary table temp_t(id int primary key)engine=innodb;
```
该sql流程：MySQL 要给这个 InnoDB 表创建一个 frm 文件保存表结构定义，还要有地方保存表数据。这个 frm 文件放在临时文件目录下，文件名的后缀是.frm，前缀是“#sql{进程 id}_{线程 id}_ 序列号”
可以根据命名可以看出mysql认为临时表名T1和普通表名T1是不一样的

普通表和临时表通过table_def_key区分
- 普通表的 table_def_key 的值是由“库名 + 表名”得到的
- 对于临时表，table_def_key 在“库名 + 表名”基础上，又加入了“server_id+thread_id”

**临时表和主备复制**
```java
create table t_normal(id int primary key, c int)engine=innodb;/*Q1*/
create temporary table temp_t like t_normal;/*Q2*/
insert into temp_t values(1,1);/*Q3*/
insert into t_normal select * from temp_t;/*Q4*/
```
如果关于临时表的操作都不记录，那么在备库执行到 insert into t_normal 的时候，就会报错“表 temp_t 不存在”。
只在 binlog_format=statment/mixed 的时候，binlog 中才会记录临时表的操作
创建临时表的语句会传到备库执行，主库在线程退出的时候，会自动删除临时表，但备库同步线程是持续在运行的，需要在主库上再写一个 DROP TEMPORARY TABLE 传给备库执行。

主库上不同的线程创建同名的临时表是没关系的，但是备库的应用日志线程是共用的，但不会导致同步线程报错，因为MySQL 在记录 binlog 的时候，会把主库执行这个语句的线程 id 写到 binlog 中，备库利用这个线程 id 来构造临时表的 table_def_key

#### 临时内存表
- 如果语句执行过程可以一边读数据，一边直接得到结果，是不需要额外的内存，来保存中间结果；
- join_buffer 是无序数组，sort_buffer 是有序数组，临时表是二维表结构；
- 如果执行逻辑需要用到二维表特性，就会优先考虑使用临时表。比如union需要用到唯一索引约束， group by 还需要用到另外一个字段来存累积计数

参数 tmp_table_size控制内存临时表大小的，发现内存临时表大小到达了上限，这时候会把内存临时表转成磁盘临时表

group by 优化方法 -- 索引
论是使用内存临时表还是磁盘临时表，group by 逻辑都需要构造一个带唯一索引的表，执行代价都是比较高的。如果表的数据量比较大，上面这个 group by 语句执行起来就会很慢由于每一行的 id%100 的结果是无序的，需要有一个临时表。如果保证出现的数据是有序，就不需要临时表，也不需要额外排序。具体为创建一个列 z，然后在 z 列上创建一个索引,如：
```java
alter table t1 add column z int generated always as(id % 100), add index(z);
```

group by 优化方法 -- 直接排序 
果一个group by语句中需要放到临时表上的数据量特别大，使用SQL_BIG_RESULT，告诉mysql直接用磁盘临时表。

### innodb和memory
**数据组织方式**

- InnoDB 引擎把数据放在主键索引上，其他索引上保存的是主键id。这种方式，我们称之为为**索引组织表**。
- 而Memory引擎采用的是把数据单独存放，索引上保存数据位置的数据组织形式，我们称之为**堆组织表**。

**区别**

1. InnoDB 表的数据总是有序存放的，而内存表的数据就是按照写入顺序存放的；
2. 当数据文件有空洞的时候，InnoDB表在插入新数据的时候，为了保证数据有序性，只能在固定的位置写入新值，而内存表找到空位就可以插入新值
3. 数据位置发生变化的时候，InnoDB表只需要修改主键索引，而内存表需要修改所有索引
4. InnoDB 表用主键索引查询时需要走一次索引查找，用普通索引查询的时候，需要走两次索引查找。而内存表没有这个区别，所有索引的“地位”都是相同的。
5. InnoDB 支持变长数据类型，不同记录的长度可能不同；内存表不支持 Blob 和 Text 字段，并且即使定义了 varchar(N)，实际也当作 char(N)，也就是固定长度字符串来存储，因此内存表的每行数据长度相同。

**内存表使用B-tree索引**
```java
alter table t1 add index a_btree_index using btree (id);
```

**缺点**
- 锁粒度问题：内存表不支持行锁，只支持表锁。
- 数据持久性问题：数据放在内存中，数据库重启，内存表也会被清空（特别是在主备模式下的，会导致主备同步失败，以及一系列异常）

### 自增主键

**自增值保存**

- MyISAM 引擎的自增值保存在数据文件中。
- innoDB 引擎的自增值，保存在内存里（MySQL 5.7 及之前的版本,每次重启后，第一次打开表的时候，都会去找自增值的最大值max(id)），MySQL8.0版本后，才有了“自增值持久化（发生重启，表的自增值可以恢复为MySQL重启前的值,记录在了 redo log） 

**自增值修改机制**
- 插入数据时 id 字段指定为 0、null 或未指定值，当前的 AUTO_INCREMENT 值填到自增字段
- 插入数据时 id字段指定了具体的值，就直接使用指定的值
- 插入的值是 X，当前的自增值是 Y
    - 如果 X<Y,那么这个表自增值不变
    - 如果X>=Y，把当前值修改为新的自增值

**新的自增值生成算法**
从 auto_increment_offset开始，以auto_increment_increment 为步长，持续叠加，直到找到第一个大于 X 的值，作为新的自增值。

**自增主键id不连续**
- 插入一条记录的唯一字段重复时，会造成一个自增主键id不连续
- 回滚也会产生这种现象

**自增值为什么不能回退**
如果两个并行执行的事务，在申请自增值后有个事务失败了，如果回退，那么接下来的事务会出现主键重复的冲突

MySQL 5.1.22新增参数 innodb_autoinc_lock_mode

1. 这个参数的值被设置为0时，语句执行结束后才释放锁
2. 这个参数的值被设置为 1 时，普通 insert 语句，自增锁在申请之后就马上释放，insert … select 批量插入数据语句，自增锁还是要等语句结束后才被释放
3. 被设置为 2 时，所有的申请自增主键的动作都是申请后就释放锁

insert … select使用语句级的锁是为了多并发条件下单主从数据的一致性

### 怎么快速复制一张表
insert … select：适用于扫描行数和加锁范围很小
很有可能造成对源表加读锁 

**mysqldump方法**
使用mysqldump导出成一组insert语句，输出到临时文件
```java
mysqldump -h$host -P$port -u$user --add-locks=0 --no-create-info --single-transaction  --set-gtid-purged=OFF db1 t --where="a>900" --result-file=/client_tmp/t.sql
```
- –single-transaction:在导出数据不需要对表 db1.t 加表锁
- –add-locks 设置为0，表示在输出的文件结果里，不增加"LOCK TABLES t WRITE;"
- no-create-info 的意思是，不需要导出表结构
- set-gtid-purged=off 表示的是，不输出跟GTID 相关的信息
- result-file 指定了输出文件的路径，其中client 表示生成的文件是在客户端机器上的

**导出 CSV 文件**
直接将结果导出成.csv文件,将查询结果导出到服务端本地目录
```java
select * from db1.t where a>900 into outfile '/server_tmp/t.csv';
```
注意：结果保存在服务端2into outfile 指定了文件的生成位置（/server_tmp/），这个位置必须受参数secure_file_priv的限制3条命令不会帮你覆盖文件4这条命令生成的文本文件中，原则上一个数据行对应文本文件的一行 

**将数据导入到目标表**
```java
load data infile '/server_tmp/t.csv' into table db2.t;
```

由于 /server_tmp/t.csv文件只保存在主库所在的主机上，如果只是把这条语句原文写到 binlog 中，在备库执行的时候，备库的本地机器上没有这个文件，就会导致主备同步停止

**物理拷贝方法**
可传输表空间:

1. 执行 create table r like t，创建一个相同表结构的空表
2. 执行 alter table r discard tablespace，这时候 r.ibd 文件会被删除
3. 执行 flush table t for export，这时db1 目录下会生成一个 t.cfg 文件(db1.t 整个表处于只读状态，直到执行第5步，unlock table)
4. 在 db1 目录下执行 cp t.cfg r.cfg; cp t.ibd r.ibd；这两个命令（拷贝得到的两个文件，MySQL 进程要有读写权限)
5. 执行 unlock tables，t.cfg 文件会被删除
6. 执行 alter table r import tablespace，将这个 r.ibd 文件作为表 r 的新的表空间，由于这个文件的数据内容和 t.ibd 是相同的，所以表 r 中就有了和表 t 相同的数据（为了让文件里的表空间 id 和数据字典中的一致，会修改r.ibd 的表空间 id）
 
#### grant赋权和flush privileges
```java
create user 'ua'@'%' identified by 'pa';
```
1. 磁盘上，往 mysql.user 表里插入一行,不指定权限
2. 内存里，往数组 acl_users 里插入一个acl_user 对象，这个对象的 access 字段值为0

**全局权限**
```java
grant all privileges on *.* to 'ua'@'%' with grant option;
```
1. 磁盘上，将 mysql.user 表里，用户’ua’@’%''这一行的所有表示权限的字段的值都修改为‘Y’
2. 内存里，从数组 acl_users 中找到这个用户对应的对象，将 access 值（权限位）修改为二进制的“全 1”。

**回收权限**
```java
revoke all privileges on *.* from 'ua'@'%';
```
1. 磁盘上，将 mysql.user 表里，用户’ua’@’%''这一行的所有表示权限的字段的值都修改为‘N’
2. 内存里，从数组 acl_users 中找到这个用户对应的对象，将 access 值（权限位）修改为二进制的“全 0”。

**DB权限**
```java
grant all privileges on db1.* to 'ua'@'%' with grant option;
```
1. 磁盘上，往 mysql.db 表中插入了一行记录所有权限位字段设置为“Y”；
2. 内存里，增加一个对象到数组 acl_dbs 中，这个对象的权限位为0

grant 操作对于已经存在的连接的影响，在全局权限和基于 db 的权限效果是不同的

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/52.png) 

set global sync_binlog需要 super 权限
因为 super 是全局权限，权限信息在线程对象中，而 T3revoke 操作影响不到这个线程对象。
acl_dbs 是一个全局数组,db 权限都用这个数组,T5
revoke 操作马上就会影响到 session B

**表权限和列权限**
```java
create table db1.t1(id int, a int);

grant all privileges on db1.t1 to 'ua'@'%' with grant option;
GRANT SELECT(id), INSERT (id,a) ON mydb.mytbl TO 'ua'@'%' with grant option;
```
这两个权限每次 grant 的时候都会修改数据表，也会同步修改内存中的 hash 结构,也会马上影响到已经存在的连接

flush privileges命令会清空acl_users数组，然后从mysql.user表中读取数据重新加载，重新构造一个 acl_users 数组
 
 ### 分区表
**分区表的引擎层行为**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/53.png) 
session B 的第一个 insert 语句是可以执行成功的。这是因为，对于引擎来说，p_2018 和 p_2019是两个不同的表，也就是说 2017-4-1 的下一个记录并不是 2018-4-1，而是 p_2018 分区的 supremum 

**分区策略**
每当第一次访问一个分区表的时候，MySQL 需要把所有的分区都访问一遍
MyISAM 通用分区策略每次访问分区都由 server 层控制 
5.7.9 InnoDB 引擎引入了本地分区策略，是在 InnoDB 内部自己管理打开分区的行为
8.0 版本开始只有InnoDB和NDB这两个引擎支持了本地分区策略

**分区表的 server 层行为**
从 server 层看的话，一个分区表就只是一个表
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/54.png) 
sessionA持有MDL锁，导致了 session B 的 alter 语句被堵住 

**分区表的应用场景**
优势是对业务透明，相对于用户分表来说，使用分区表的业务代码更简洁
如果一项业务跑的时间足够长，往往就会有根据时间删除历史数据的需求，可以直接通过 alter table t drop partition删掉分区，速度快、对系统影响小
 