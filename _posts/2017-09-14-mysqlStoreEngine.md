---
layout: post
title: 'Mysql存储引擎-myisam和innodb'
subtitle: 'myiasm和innodb区别'
date: 2017-09-14
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---


- Innodb不支持**全文索引（fulltext）**
- Innodb表中不存储数据**具体行数**，所以myisam的select count(*)比innodb快
- 我们都知道Innodb是**行级锁**，但是某些情况下也是会**锁表**的比如模糊查询
- Innodb最主要差别就是Innodb支持**事务处理和外键和行级锁**
- MyIsam**拷贝**整张表，直接拷贝对应数据库下的 frm myd myi文件即可

- MyIsam和Innodb都是使用**B+tree**作为索引结构

- Innodb 不建议使用**过长的字段**作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大;也不建议使用**非单调字段**作为主键，非单调主键在插入B+树时会不断的进行旋转，分裂，会很低效

特点：

- InnoDB：存储限制64TB，支持事务，树索引，哈希索引，数据缓存，外键，表锁，行锁。适合需更新多场景
- MylSAM：存储限制256TB，全文索引，树索引。适合select和insert多的场景
- MEMORY：存储限制RAM，哈希索引，树索引。适合临时存储，速度要求快数据场景
- MERGE：多MyIsam表集合，相当于分表，但是一条sql查询所有表
 

**全文索引**：建立词库，数据中查找每个词条出现频率，位置，并顺序归纳为索引。