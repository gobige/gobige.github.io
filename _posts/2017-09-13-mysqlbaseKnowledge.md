---
layout: post
title: 'Mysql一些基础知识'
subtitle: 'Mysql一些基础知识'
date: 2017-09-13
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---


## 前言
mysql 一些常用的指令，语法，以及一些库函数

## SQLite Mysql PostgreSQL
SQLite的优点
**占用空间小**：占用不到**600KiB**的空间。 **无需任何外部依赖**项即可运行SQLite。
用户友好： “**零配置**”数据库，SQLite不会作为服务器进程运行， 永远都**不需要停止，启动或重新启动**，不需要任何需要管理的配置文件。 
可移植：整个SQLite数据库存储在**单个文件**中。该文件可位于**任何位置**，可通过可移动媒体或文件传输协议共享

SQLite的缺点
**有限的并发性**：多个进程可以同时访问和查询，但**任何给定时间只有一个进程可以对数据库进行更改**。  
没有用户管理：SQLite直接读取和写入普通磁盘文件唯一适用的访问权限是**基础操作系统的典型访问权限**。
**安全性**：使用服务器的数据库引擎比无服务器的数据库（如SQLite）可以更好地保护客户端应用程序中的错误（并发，锁）。

适用场景
**嵌入式**应用程序，**测试**，磁盘访问替换

PostgreSQL的优点
SQL合规性：比MySQL还要**严格遵守SQL标准**
开源和社区驱动：完全**开放源代码**的项目
可**扩展**：通过其目录驱动的操作及其对动态加载的使用

PostgreSQL的缺点
内存性能：对于每个新的客户端**连接**，PostgreSQL都会派生一个大约**10MB新进程**。
流行度：帮助管理PostgreSQL数据库的第**三方工具仍然较少**。经验丰富的Postgres数据库的数据库管理员也少

**速度,复杂的复制上面较弱于Mysql**

**NULL**

大多数数据库中NULL和空字符串是区别对待的，空字符串和NULL都是一种**值**，Oracle中所有的空字符串都会自动转换成NULL
SELECT * FROM SOME_TABLE WHERE 1 = NULL；查询数据为0

mysql中 NULL 就是 NULL，是**【未知】**，1 = NULL 1 != NULL,1就是1，数据库不知道1是不是等于【未知】

## 数据库操作指令

**显示mysql版本**
select version;
 
**连接数据库**
mysql -h 主机地址 -u 用户名 －p 用户密码

**修改密码**
mysqladmin -u 用户名 -p 旧密码 password 新密码

**查看mysql支持的字符集**
show CHARACTER set   

**查看支持校对的完整列表**
show COLLATION; 

**查看使用的字符集**
SHOW VARIABLES LIKE 'character%'  

**查看使用的校对**
SHOW VARIABLES LIKE 'collation%'  

**管理用户**
USE mysql;

SELECT user FROM USER;

**创建用户**
CREATE USER yates IDENTIFIED BY 'yates'

**重命名用户**
RENAME USER yates to guest;

**删除用户**
DROP USER guest；

**显示用户权限**
SHOW GRANTS FOR yates;

**赋权限**
GRANT SELECT ON user_body_info.* TO yates

**撤权限** 
REVOKE SELECT ON user_body_info.* TO yates

**修改用户密码**
 SET PASSWORD FOR yates = PASSWORD('123456');


**创建数据库**
create database 数据库名;

**删除数据库** 
drop database 数据库名;

**显示所有数据库**
Show databases;

**使用某数据库**
use 数据库名;

**创建表**
create table 表名 (字段名1 类型1 ,字段名 类型);

**显示所有表**
Show tables;

**显示表结构**
desc 表名;

**删除表结构和内容**
drop 表名; 

**删除表内容**
truncate 表名(效率高)；delete 表名；

**插入数据**
insert into MyClass values(1,'Tom',96.45),(2,'Joan',82.99), (2,'Wang', 96.59);

**表复制语句**

**insert** table1 **select** from table2：table1已经存在
**select into** table1 from table2：table1不存在

**查询数据**
select * from MyClass order by id limit 0,2;

**删除数据**
delete from MyClass where id=1;

**修改数据**
update 表名 set 字段=新值,… where 条件; 
update ignore 发生错误更新继续

**修改表结构**
- alter table 表名 add 字段 类型
- alter table 表名 modify COLUMN 字段 字段类型定义
- alter table 表名 change 原字段名  新字段名 字段类型
- alter table 表名 drop column 字段名
 

## **视图**

**视图规则和限制**

- 唯一命名
- 创建没有数量限制 
- 可以嵌套
- 可以使用order by
- 不可以索引
- 和表一起用


**优点**

- 隐藏复杂sql
- 格式化检索数据

**视图的更新**

如果视图定义中有 分组，联结，子查询，并，聚集函数，distinct，导出计算列不能进行更新

## JOIN的用法 
- join对查询条件如果是多对多的关系,则进行笛卡尔乘积运算
- inner join : 内联两张关联表,将符合条件的两张关联表中记录进行展示
- left join : 左连接查询会返回左表中所有记录，不管右表中有没有关联的数据
- right join : 右连接查询会返回右表中所有记录，不管左表中有没有关联的数据, 
- FULL OUTER JOIN   外连接查询能返回左右表里的所有记录，其中左右表里能关联起来的记录被连接后返回
- LEFT JOIN EXCLUDING INNER JOIN   返回左表有但右表没有关联数据的记录集
- RIGHT JOIN EXCLUDING INNER JOIN   返回右表有但左表没有关联数据的记录集
- FULL OUTER JOIN EXCLUDING INNER JOIN 返回左表和右表里没有相互关联的记录集。

## **存储过程**

**Why use produce**

- 简化复杂操作
- 封装处理过程
- 简化对变动的管理
- 提高性能

**create procedure**
CREATE PROCEDURE PROCEDURE1(OUT VAR1 DECIMAL(8,2),IN VAR2,INOUT VAR3) BEGIN SELECT * from table END;


**Delimiter**

告诉命令行使用//作为新的结束分隔符

- **使用存储过程**  CALL PROCEDURE1(@VAR1,VAR2,@VAR3)
- **删除存储过程** DROP PROCEDURE PROCEDURE1 IF EXISTS;
- **检查存储过程**  SHOW CREATE PROCEDURE PRO1;

## **游标**

只可用在存储过程中

**创建游标**

CREATE PROCEDURE pro1 ()BEGIN DECLARE cur1 CURSOR FOR SELECT *FROM user_body_info;END;

**打开游标 **

OPEN cur1

**检索游标内容**

Fetch 

## **触发器**
每个表支持最多6个

**创建触发器 **

CREATE TRIGGER tri1 AFTER INSERT ON user_body_info FOR EACH ROW SELECT 'insert success aytes'

## **事务**

**开启事务 **

START TRANSACTION;

**回退 **

ROLLBACK

**提交事务** 

commit

**保留点** 

SAVEPOINT p1


## **显示mysql服务器状态**

show status;

https://www.cnblogs.com/zuxing/articles/7761262.html

**显示当前时间**
select now();

**显示年月日**
- SELECT YEAR(CURRENT_DATE);
- SELECT DAYOFMONTH(CURRENT_DATE); 
- SELECT MONTH(CURRENT_DATE);

**显示字符串**
SELECT "welecome to my blog!";

**计算器**
select ((4 * 4) / 10 ) + 25;

## **字符串操作相关函数**

**拼接**

select **CONCAT**(f_name, " ", l_name) from table

注意：如有任何一个参数为 NULL ，则返回值为NULL

**CONCAT_WS**(separator,str1,str2,...)

注意：第一个参数是其它参数的分隔符。如果分隔符为 NULL ，则结果为 NULL

**REPEAT**(str ,count)

由重复的字符串str组成的字符串,字符串str的数目等于count
注意：若 count <= 0, 则返回一个空字符串。若str 或 count 为 NULL ，则返回 NULL

**字符串处理**

LTrim ---左去除空格
RTrim---右去除空格
Left(str,len) ---返回str左边几个字符
Right(str,len) ---返回str右边几个字符
LOCATE(substr,str)  ---找出串的一个子串
Upper()----文本转大写
Lower()----文本转小写
Soundex 返回串的soundex值

**统计**

- **CHAR_LENGTH(str)**:不管汉字还是数字或者是字母都算是一个字符

- **Length(str)**:是计算字段的长度一个汉字是算三个字符,一个数字或字母算一个字符


## 日期函数

- **AddDate()**  增加一个日期
- **AddTime()**  增加一个时间
- **CurDate()**   返回当前日期
- **CurTime()**   返回当前时间
- **Date(time)**  返回当前时间日期部分
- **Datediff(tim1,tim2)** 两个时间日期之差
- **DATE_FORMAT(date,format)** 格式化时间
- **Day()**返回日期天数部分
- **dayOFweek**  对于一个日期，返回对应的星期几
- **Hour()**  返回小时部分
- **Minute()** 返回分钟部分
- **Month()** 返回月份部分
- **Now()** 返回当前时间
- **Second()**  返回时间秒部分
- **Time()**  返回一个日期时间时间部分
- **Year()**  返回一个日期年份部分

## 数值处理函数

- **Abs()**  返回绝对值
- **Cos()**  返回角度余弦
- **Exp()**  返回一个数的指数
- **Mod()**  返回除操作的余数
- **Pi()**  返回圆周率
- **Rand()**  返回随机数
- **Sin()**  返回角度正弦
- **Sqrt()**  返回一个数的平方根
- **Tan()**   返回一个角度的正切


## 聚合函数
- **AVG()**  返回某列平均值  忽略null
- **Count()**  查询行数 指定列忽略 *不忽略
- **MAX()**  返回某列最大值   忽略null
- **Min()**  某列最小  忽略null
- **Sum()** 某列值和  忽略null

- **Group by** 分组包含null，和聚集函数一起使用  
- **having** 过滤分组 ，where过滤行

**没有联结条件**的表关系返回的结果是**笛卡尔集**

**自连 左连 右连 内联 外联**

**导出整个数据库(导出文件默认是存在mysql\bin目录下)**

mysqldump -u 用户名 -p 数据库名 > 导出的文件名

**导出一个表**

mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名

**导出一个数据库结构**

mysqldump -u user_name -p -d –add-drop-table database_name > outfile_name.sql
    -d 没有数据 –add-drop-table 在每个create语句之前增加一个drop table


**带语言参数导出**

mysqldump -u root -p –default-character-set=latin1 –set-charset=gbk –skip-opt database_name > outfile_name.sql
