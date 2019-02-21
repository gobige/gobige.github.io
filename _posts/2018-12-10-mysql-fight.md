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

那么mysql通过什么值得binlog是否完整的呢?1statement的binlog,最后会有commit;2row格式的binlog,最后会有一个XID event. mysql5.6.2版本后还引入了binlog-checksum参数,用于验证内容正确性
redolog和binlog通过什么关联?通过XID标识关联,binlog在写入后就会从从库取出来,所以主库也要通过redolog提交这个事务,从而保证数据一致性.


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
有时我们会发现我们是哟功能delete语句删除了数据之后,发现表空间大小没有变化,这是由于delete语句只是将删除的记录置为**可复用**状态,磁盘文件大小并不会缩小,看起来就像是**空洞**,不仅删除会造成空洞,插入数据也会,如果数据不是按照索引递增顺序插入,而是随机插入的,就可能造成索引的**数据页分裂**.

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
1. 创建临时表,是哟功能memory引擎,表里有两个字段,一个是R,一个W.这个表没有索引
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
1. 条件字段**函数的使用**会导致索引字段的有序性遭到破坏，于是优化器在sql优化时会放弃走该字段的树搜索功能，造成sql语句执行慢
2. 数据类型转换规则：字符串和数字作比较的话，是将**字符串转换成数字**。因此mysql中隐式转换也会造成sql语句执行慢
3. **隐式字符编码**转换，如果两种表的字符集使用不统一，那么在两张变做关联查询时，被驱动表的索引字段有可能作隐式转换，从而导致对被驱动表作全表扫描，sql执行慢。解决方案（1统一编码字符集，2通过修改关联表之间连接顺序）
4. **锁表**可能导致查询语句的长时间不返回，通过show processlist命令，查看当前语句的状态，如果看到**Waiting for table metadata lock**的状态，说明存在线程正在表T上请求或持有**MDL写锁**，该查询语句被阻塞,如图：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/8.png)
针对这种阻塞的处理方式时找到持有MDL写锁的线程，然后kill掉，那么如何找到该线程呢。我们看到**sleep指令**是无法查找的，只有通过performance_schema和sys系统来查询sys.schema_table_lock_waits表，就可以查找出阻塞process_id（Mysql需开启performance_schema=on）

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/9.png)

5. **等flush**过程中，会导致查询语句阻塞，通过如果对一个表做flush操作指令为
```java
// 只关闭表T 
flush tables t with read lock;

// 关闭所有表 
flush tables with read lock;
```
这两个指令本来是很快的，除非它们也被别的线程阻塞。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/10.png)

6. **等行锁**
```java
select * from t where id = 1 lock in share mode;
```
当我们执行这句查询时，id为1的记录会被加读锁，如果这时有一个事务在这行记录持有一个写锁，那么该查询语句就会阻塞，通过查询
```java
select * from t sys.innodb_lock_waits where locked_table=`'test'.'t'`
```
查询堵塞的源头，如图：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/11.png)


那么除了锁造成查询sql慢以外还有没有其他的原因呢

**一致性读**

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/12.png)

对比上面两个sql，前者是普通的一致性读，后者是当前读，会用到读锁
如图：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/mysql/13.png)

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

可重复读事务条件下的加锁规则

**原则**
1. 加锁基本单位是next-key lock，next=key lock是**前开后闭区间**
2. 查找过程中访问到的对象才会加锁

**优化**
1. 索引上的**等值查询**，给**唯一索引加锁**的时候，next-key lock退化为**行锁**
2. 索引上的**等值查询**，向**右遍历**时**最后一个值不满足**等值条件的时候，next-key lock退化为**间隙锁**

**bug**
**唯一索引**上的范围查询会访问到不满足条件的第一个值为止。


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
 
### 如何判断数据出问题

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