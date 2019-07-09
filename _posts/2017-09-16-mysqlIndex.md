---
layout: post
title: 'MySQL索引'
subtitle: 'mysql一些索引相关知识'
date: 2017-09-16
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---

### 数据库锁类型
排它锁：数据对象被加锁后，其他事务不能对其读取和修改
共享锁：数据被加锁后，其他事务可以对其读取，但是不能进行修改操作

### 死锁
- 第一种情况
    - 线程1访问表A，同时还需要操作表B，线程2访问表B，同时还需要操作表A；这时表A，表B都被同事锁定，资源无法释放，造成死锁
    - 解决方案:同时锁定多个资源时，按照**相同顺序**获取；不能获取资源时，主动释放已经获取的资源
    
- 第二种情况
    - A事务先查询一条记录，然后修改这条记录，共享锁需要上升到排它锁，而另外一个B事务正在修改这条记录，A需要B释放排它锁才能进行下一步，而B因为A有共享锁需等待A释放共享锁
    - 解决方案:表中加入version字段，使用乐观锁来解决（每次记录修改时version + 1，同时与数据库中记录的version进行对比，如果写入的version比当前数据库中version小，则失败）

- 第三种情况
    - 复杂的sql或者 不规范的sql造成的全表扫描，锁表行为，
    - 解决方案：优化sql

**死锁案例**
线程1：SELECT * FROM `user` WHERE id in (3,5) FOR UPDATE;
线程2：SELECT * FROM `user` WHERE id in (4,5,6) FOR UPDATE; id为5的用户已经被锁住了，所以这里会报错超时

**存在的行会进行行锁，不存在的行会导致范围锁**
线程1：SELECT * FROM `user` WHERE id in (1111) FOR UPDATE; 如果1111是不存在，且比user表最大的id大，则**锁1111到无穷大区域**；如果1111是不存在，且比user表最小的id小，则**锁1111到无穷小区域**；如果1111不存在，且是在某个区间，则**锁该区间**的行锁


### 聚集索引和非聚集索引
- 聚集索引：索引的键值逻辑顺序决定了表数据行的物理存储顺序，比如主键索引;叶节点data域存放的是**数据**
- 非聚集索引：存储的键值逻辑连续，但是在表数据行物理存储顺序上不一定连续的索引;叶节点data域存放的是**数据记录的地址**
 
### 索引失效
- 隐式转换导致索引失效。最常见的是使用varchar类型存了像订单号这种纯数字的数据，但是sql查询时使用的时候传入number类型的情况，包括时间等类型
    - select * from order where orderNumber = 1231231231    错误
    - select * from order where orderNumber = '1231231231'  正确

- 对索引字段进行运算操作（+，-，*，/，！）
- 使用 = null 而不是正确使用is null或者in not null
- 使用复合索引时查询条件先后顺序不对应
- 使用<>,not in, not exist,!=，%queryData
- sql语句中使用双引号

### 索引类型
mysql索引类型壳分为B-tree，hash，fulltext，r-tree