---
layout: post
title: 'Mysql一些基础知识'
subtitle: 'myiasm和innodb区别'
date: 2017-09-13
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---

### 前言
mysql 一些常用的指令，语法，以及一些库函数
 
**连接数据库**
mysql -h 主机地址 -u 用户名 －p 用户密码

**修改密码**
mysqladmin -u 用户名 -p 旧密码 password 新密码

**创建数据库**
create database 数据库名;

**删除数据库** 
drop database 数据库名;

**创建表**
create table 表名 (字段名1 类型1 ,字段名 类型);

**显示表结构**
desc 表名;

**删除表结构和内容**
drop 表名; 

**删除表内容**
truncate 表名(效率高)；delete 表名；

**插入数据**
insert into MyClass values(1,'Tom',96.45),(2,'Joan',82.99), (2,'Wang', 96.59);

**查询数据**
select * from MyClass order by id limit 0,2;

**删除数据**
delete from MyClass where id=1;

**修改数据**
update 表名 set 字段=新值,… where 条件; 

**修改表结构**
alter table 表名 add 字段 类型
alter table 表名 modify COLUMN 字段 字段类型定义
alter table 表名 change 原字段名  新字段名 字段类型
alter table 表名 drop column 字段名

**显示当前版本**
SELECT VERSION();

**显示mysql服务器状态**
show status;
https://www.cnblogs.com/zuxing/articles/7761262.html

**显示当前时间**
select now();

**显示年月日**
SELECT DAYOFMONTH(CURRENT_DATE); 
SELECT MONTH(CURRENT_DATE);
SELECT YEAR(CURRENT_DATE);

**显示字符串**
SELECT "welecome to my blog!";

**计算器**
select ((4 * 4) / 10 ) + 25;

**字符串操作相关函数**
**拼接**
select CONCAT(f_name, " ", l_name) from table
注意：如有任何一个参数为 NULL ，则返回值为NULL

CONCAT_WS(separator,str1,str2,...)
注意：第一个参数是其它参数的分隔符。如果分隔符为 NULL ，则结果为 NULL

REPEAT(str ,count)
由重复的字符串str组成的字符串,字符串str的数目等于count
注意：若 count <= 0, 则返回一个空字符串。若str 或 count 为 NULL ，则返回 NULL

**统计**
CHAR_LENGTH(str)
不管汉字还是数字或者是字母都算是一个字符

Length(str)
是计算字段的长度一个汉字是算三个字符,一个数字或字母算一个字符

**导出整个数据库(导出文件默认是存在mysql\bin目录下)**
mysqldump -u 用户名 -p 数据库名 > 导出的文件名

**导出一个表**
mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名

**导出一个数据库结构**
mysqldump -u user_name -p -d –add-drop-table database_name > outfile_name.sql
    -d 没有数据 –add-drop-table 在每个create语句之前增加一个drop table
****

**带语言参数导出**
mysqldump -uroot -p –default-character-set=latin1 –set-charset=gbk –skip-opt database_name > outfile_name.sql
