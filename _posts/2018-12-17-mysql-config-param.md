---
layout: post
title: 'mysql配置参数'
subtitle: 'mysql配置参数'
date: 2018-12-17
categories: mysql
author: yates
cover: ''
tags: mysql
---

## mysql逻辑架构

max_connections：MySQL的最大连接数，每个连接提供连接缓冲区，就会开销越多的内存

back_log：MySQL的连接数据达到max_connections时，新的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈数量即back_log，如果等待连接的数量超过back_log，将不被接受连接资源

wait_timeout：指的是MySQL关闭一个**非交互的连接**之前所需要等待的秒数。

interative_timeout：指的是关闭一个**交互的连接**之前所需要等待的秒数。

query_cache_size：查询缓存，MySQL将查询结果存放在缓冲区

query_cache_type：缓存类型   0：OFF 相当于禁用了   1：ON 将缓存所有结果  2：DENAND  则只缓存select语句中通过SQL_CACHE

sort_buffer_size：使用order by排序使用内存，排序超过此设置会使用row_id排序

thread_cache_size：服务器线程缓存，这个值表示可以重新利用保存在缓存中的线程数量

innodb_buffer_pool_size：可作为数据和索引缓存的内存大小

innodb_flush_log_at_trx_commit：redo_log事务flush磁盘策略，据说 1和N时影响区别很大

innodb_thread_concurrency：innodb线程并发数

innodb_log_buffer_size：redo_log日志文件所用的内存大小

innodb_log_file_size：redo_log文件数

innodb_log_files_in_group：循环覆盖刷盘文件数量