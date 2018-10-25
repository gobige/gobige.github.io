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


- Innodb不支持全文索引（fulltext）
- Innodb表中不存储数据具体行数，所以myisam的select count(*)比innodb快
- 我们都知道Innodb是行级锁，但是某些情况下也是会锁表的比如模糊查询
- Innodb最主要差别就是Innodb支持事务处理和外键和行级锁
- MyIsam拷贝整张表，直接拷贝对应数据库下的 frm myd myi文件即可

- MyIsam和Innodb都是使用**B+tree**作为索引结构，myisam的叶节点data域存放的是**数据记录的地址**，这种索引方式也叫作**非聚集索引**;而innodb的叶节点data域存放的是**数据**，这种索引叫做**聚集索引**;myisam的**主索引和辅助索引结构没有任何区别**，主索引要求key是唯一的，辅助索引key可以不唯一；innodb**必须**有一个主键作为索引，所有**辅助索引都是引用主键索引的key**作为data域，这种索引叫做**聚集索引**
- Innodb 不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大;也不建议使用非单调字段作为主键，非单调主键在插入B+树时会不断的进行旋转，分裂，会很低效

!(关于innodb索引的原理)[http://www.admin10000.com/document/5372.html]