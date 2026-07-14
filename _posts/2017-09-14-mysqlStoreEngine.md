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


**大多数应用场景不再使用 MyISAM**：
1. 锁粒度：表锁（MyISAM）vs 行锁（InnoDB）—— 高并发的生死线
- 这是最直接影响系统吞吐量的性能瓶颈。
 
2. 事务支持：无（MyISAM）vs 完善的 ACID（InnoDB）
- 现代业务系统（特别是金融、电商、信贷等资金或核心业务）对数据一致性有着极高的要求。

3. 崩溃恢复（Crash Recovery）与可靠性
- 数据库服务器异常断电、宕机是无法绝对避免的，这就极其考验引擎的自愈能力。
   - 如果服务器在写入 MyISAM 表时突然断电，该表的数据和索引文件非常容易损坏（Corrupted）。重启后，你需要手动执行繁重的 REPAIR TABLE，且无法保证数据不丢失。
   - InnoDB 拥有 Crash-safe 能力：依靠著名的 WAL（Write-Ahead Logging，日志先行） 机制。即使数据库意外断电，重启后 InnoDB 会自动读取 Redo Log 进行前滚和回滚，自动恢复到一致的状态，确保零数据丢失。
4. 缓存机制：只缓存索引 vs 缓存数据+索引
- MyISAM 的缓存策略：它的 Buffer Pool 只缓存索引（.MYI 文件），不缓存具体的数据。当读取具体数据行时，每一次都需要直接去读磁盘上的 .MYD 文件。这极大地依赖操作系统的文件系统缓存，性能上限很低。
 
5. 外键与约束支持
- MyISAM 不支持外键（Foreign Key）：InnoDB 支持外键，能从数据库底层强力保障数据表之间的关联一致性（虽然现在大厂互联网开发规范中更提倡“在业务代码中控制关联关系，少用物理外键”，但底层支持仍然是重要的技术基石）。

特点：

- InnoDB：存储限制64TB，支持事务，树索引，哈希索引，数据缓存，外键，表锁，行锁。适合需更新多场景
- MylSAM：存储限制256TB，全文索引，树索引。适合select和insert多的场景
- MEMORY：存储限制RAM，哈希索引，树索引。适合临时存储，速度要求快数据场景
- MERGE：多MyIsam表集合，相当于分表，但是一条sql查询所有表
 

**全文索引**：建立词库，数据中查找每个词条出现频率，位置，并顺序归纳为索引。
