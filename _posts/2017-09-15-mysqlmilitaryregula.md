---
layout: post
title: 'MySQL军规'
subtitle: '一些mysql开发遵守的约定'
date: 2017-09-15
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---

### 核心军规


- 尽量不在数据库做运算
- 控制单表数据量 纯INT不超过10M条，含Char不超过5M条
- 保持表身段苗条
- 平衡范式和冗余
- 拒绝大SQL，复杂事务，大批量任务

### 字段类军规

- 用好数值字段，尽量简化字段位数
- 把字符转化为数字
- 优先使用Enum或Set
- 避免使用Null字段
- 少用并拆封Text/Blob
- 不在数据库中存图片

### 索引类军规

- 谨慎合理添加索引
- 字符字段必须建立前缀索引?
- 不在索引列做运算
- 自增列或全局ID做InnoDB主键
- 尽量不用外键

### SQL类军规

- SQL尽可能简单
- 保持事务连接短小
- 尽可能避免使用SP/Trigger/Function
- 尽量不用Select *
- 改写Or为IN()
- 改写Or为Union
- 避免负向查询和%前缀模糊查询
- Count不要使用在可Null的字段上面
- 减少Count(*)
- Limit高效分页，SELECT * FROM message WHERE id > 9527 (or sub select) limit 10
- 使用Union ALL 而不用Union
- 分解链接，保证高并发
- Group By 去除排序
- 同数据类型的列值比较
- Load Data导入数据，比Insert快20倍
- 打散大批量更新，尽量凌晨操作

### 约定类军规

- 隔离线上线下
- 禁止未经DBA认证的子查询
- 永远不在程序段显式加锁
- 表字符集统一使用UTF8MB4