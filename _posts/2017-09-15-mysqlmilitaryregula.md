---
layout: post
title: 'MySQL规范和优化'
subtitle: '一些mysql开发遵守的约定和总结的sql优化方案'
date: 2017-09-15
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---

### 数据库三大特性
实体：表
属性：表字段
关系：表与表之前关系

### 数据库三大范式
- **第一范式** 表中字段不可再拆分，是最小单元，确保每一列的原子性
- **第二范式** 除主键外，表中所有字段都依赖主键字段，要求每张表只描述一件事情
- **第三范式** 表中每一列只与主键直接相关而不是间接相关，一张表中只能有另一张表的id关联，不能有其他信息

### 五大约束
- primary key：主键约束
- unique：唯一性约束
- default：默认值约束
- not null：非空约束
- foreign key：外键约束（只有innodb支持外键）

### 核心军规

- 尽量不在数据库做运算
- 控制单表数据量 纯INT不超过10M条，含Char不超过5M条
- 保持表身段苗条，少而精（利于：IO高效，全表遍历，表修复快，并发，alter talble快）
- 单行不超过200Byte，不超过50个纯Int字段，不超过20个纯CHAR字段
- 平衡范式和冗余（效率优先，性能优先，适当冗余，可能会提升代码复杂度）
- 拒绝大SQL，复杂事务，大批量任务，尽量简单应用mysql

### 字段类军规

- 用好数值字段，尽量简化字段位数，使用可以存下你数据的最小的类型，使用简单的数据类型
- 尽可能使用not null 定义字段，尽量少用 大容量数据类型
- 把字符转化为数字（更高效，查询更快，占用空间小）
- 优先使用Enum或Set（可能值已知且有限）
- 避免使用Null字段
- 少用并拆封Text/Blob
- 不在数据库中存图片

### 索引类军规

- 谨慎合理添加索引（改善查询，减慢更新）
- 能不加索引尽量不加（综合评估数据密度和数据分布，不超过字段数20个百分点）
- 字符字段必须建立前缀索引
- 不在索引列做运算（无法使用索引，导致全表扫描）
- 自增列或全局ID做InnoDB主键
- 尽量不用外键（有额外开销，逐行操作，导致其他表锁定，高并发死锁）
- 优先覆盖索引
- 关联查询不同表查询字段若使用索引，字段类型应该一样

### SQL类军规

- SQL尽可能简单（一条sql单核运算，高qps大sql可能会拖死数据库；简单sql缓存命中率更高，锁表时间短，特别是myisam表级锁；多sql利用上多cpu计算）
- 保持事务连接短小（即可即用，用完关闭；短事务优先长事务）
- 尽可能避免使用SP/Trigger/Function
- 尽量不用Select *（更多cpu消耗，内存，io，带宽）
- 改写Or为IN（or时间复杂度：**O(n)**,IN时间复杂度**O(Log n)**;IN个数建议**小于20**）
- 改写Or为Union(不同字段减少merge index)
- 避免负向查询(NOT,!=,<>,!<,!>,NOT EXISTS,NOT IN,NOT LIKE)
- 避免%前缀模糊查询（B+tree，索引失效，全表扫描）
- Count不要使用在可Null的字段上面
- 减少Count(*)
- 计数统计：redis，双向更新，
- 非实时统计：尽量单独统计表，定期重算
- Limit高效分页，SELECT * FROM message WHERE id > 9527 (or sub select) limit 10（如果使用limit 9527,10 因为LIMIT 偏移量越大，越慢）
- 使用Union ALL 而不用Union（union有去重开销）
- 高并发不建议两个表以上join
- 分解链接，保证高并发（可缓存大量早期数据，可使用多个myisam表，对大表的小ID IN()，联接引用同一个表多次）
- Group By 去除排序
- 使用order by null无须排序 
- 同数据类型的列值比较
- Load Data导入数据，比Insert快20倍（成批装载比单行装载快，不用每次刷新缓存；无索引装载比索引装载比索引装载快）
- 打散大批量更新，尽量凌晨操作，执行过程sleep
- 尽量不用insert...select（延迟；同步出错）

### 约定类军规

- 隔离线上线下
- 禁止未经DBA认证的子查询
- 永远不在程序段显式加锁
- 表字符集统一使用UTF8MB4


### limit高效分页
- 子查询优化法
    - 先找出第一条数据，然后大于这条数据的id就是要获取的数据
    - 注意：数据必须是连续的，不能带有where等筛选条件
- 倒排表优化法
    - 建立索引，用一张表维护页数，高效连接得到数据
    - 注意：只适合数据固定情况，数据不能删除，维护页表困难
- 反向查找优化法
    - 当偏移超过一半记录数时候，先用排序，这样便宜就反转了
    - 注意：order by优化比较麻烦，要增加索引，索引影响数据修改效率，偏移大于总记录数一半
- limit限制优化法
    - 把limit偏移量限制低于某个数，超过这个数等于没数据

- 字段中存储日期 如果必要使用UNIX_TIMESTAMP()转化为int存储日期时间，取出时间FROM_UNIXTIME

- 字段中Ip地址 使用 INET_ATON 函数来进行转化BIGINT，取出INET_NTOA

---------------------------------------------------

- 表结构的拆分：
    * 垂直拆分 
        * 把不常用的字段单独放到一个表中
        * 把大字段独立放到一个表中
        * 把经常用的使用的字段放到一起
    * 水平拆分
        * 数据量大的

-------------------------------------------------
- 系统配置优化
    - 系统层面的优化，连接数等Mysql配置文件层面优化，
     - innodb_buffer_pool_size  缓存池大小的设定
     - innodb_buffer_pool_instances  缓冲池的个数
     - innodb_log_buffer_size  日志缓冲池大小
     - innodb_flush_log_at_trx_commit  什么时候刷新到io磁盘，0,1,2  默认1
     - innodb_read/write_io_thread 读写io线程数
     - innodb_file_per_table：每张表是否使用独立的表空间 on off

---------------------------------------------------
- 索引优化，orderby，max，min  ，数据量大，修改，增加操作少
- 联合索引 使用离散度高字段的放前面
- 索引的查找 pt duplicate-key-checker -u root -p root -h localhost;

----------------------------------------------------
**慢查日志**
1)查看mysql是否开启慢查询日志
show variables like 'slow_query_log';

2)设置没有索引的记录到慢查询日志
set global log_queries_not_using_indexes=on;

3)查看超过多长时间的sql进行记录到慢查询日志
show variables like 'long_query_time'

4)开启慢查询日志
set global slow_query_log=on;

5)设置慢查询日志记录时间
set global long_query_time=1;

6)查看慢查询日志
tail -50 D:/green/mysql-5.7.9/data/yates-PC-slow.log

---------------------------------------------------
Explain对sql执行进行分析 
- **id** id列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的
- **select_type** 查询类型 简单查询，复杂查询（简单子查询，复杂子查询（from后子查询），union查询）
	- SIMPLE, 表示此查询**不包含****UNION**查询或**子查询**
	- PRIMARY, 表示此查询是**最外层的查询**
	- UNION, 表示此查询是**UNION的第二**或**随后的查询**
	- DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询
	- UNION RESULT, UNION 的结果
	- SUBQUERY, 子查询中的第一个SELECT
	- DEPENDENT SUBQUERY: 子查询中的第一个SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果
- type 常用类型
	- system: 表中只有一条数据. 这个类型是特殊的const类型
	- const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. 
	- eq_ref: 此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果，查询效率较高
	- ref: 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了最左前缀规则索引的查询
	- range: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在<>,>,>=,<,<=,IS NULL,<=>,BETWEEN,IN()操作中
	- index: 表示全索引扫描(full index scan)
	- ALL: 表示全表扫描
- possible_keys 可能用到索引
- key 用到索引
- key_len 使用了索引字节数，可以从这个字段看出索引是否被完全使用（比如 ：复合索引中每个用到的索引key值相加）
- rows 查询需要扫描的行数
- Extra
	- Using filesort   MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果，通常表示这个查询性能不ok
	- Using index 覆盖索引扫描", 表示查询在索引树中就可查找所需数据 性能通常不错
	- Using temporary 会用到临时表，排序，分组，多表join时会出现这种情况