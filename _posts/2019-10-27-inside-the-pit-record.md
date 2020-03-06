---
layout: post
title: '那些年踩过的坑'
subtitle: '记录一些实践中踩过的坑'
date: 2019-10-27
categories: 编码
author: yates
cover: 'www.baidu.com'
tags: 编码
---

## 记录一些实践中踩过的坑
 
### 资源文件在java工程中的目录问题
这个问题是一老友问的我。情况是这样的，我们一般的java工程目录都是分为java，test,resource目录，一般资源，配置文件都是放在resource下，老友把**资源文件放在java目录**和
和java源文件放在一起，通过**代码读取配置文件中的数据，怎么也读取不到**，于是问我为什么，我看了下这个问题，然后自己试了试，确实是不行的。想了想，然后在target目录下把编译好的class
目录打开，发现并没有找到该资源文件，于是这个问题就明了了，java编译过程中只会识别java后缀的文件进行**编译**，而在之后的**类加载过程**中也只会读取class后缀的文件，所以自然在java目录下存放资源文件就不能被读取了。
而存放于resource目录下的文件会直接作为resource root移到target包下

### jpa之你真的了解jpa的执行机制吗？

**当jpa自定义sql遇上数据库某非空dateTime类型默认为'0000-00-00 00:00:00'的数据**

有这么一个需求，修改一张表的一个字段值，大概是这个样子的sql

```java
@Transactional
@Modifying
@Query(value = "UPDATE table SET colume = ?1 WHERE colume = ?3", nativeQuery = true)
int update***(String col1, Integer col2);
```

自信满满的自测该功能，咦，报错了

![此处输入图片的描述](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2019-10-18-inside-the-pit-record/1.png)

这个功能就执行了这一句sql为什么汇报这个错了？我没更新这个时间字段呀？这么诡异的事？怀着科学的态度重新编译，运行还是报同样的错，在寻求度娘的帮助下还是得不到一个有效的解释。
好吧最后的办法只能看源码了，

![此处输入图片的描述](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2019-10-18-inside-the-pit-record/2.jpg)
![此处输入图片的描述](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2019-10-18-inside-the-pit-record/3.jpg)

原来加了modifying注解的在sql执行时会flush update一次全量更新（暂时还不知道jpa为什么会这样做），这是第一个问题，这次全量更新的数据通过查询这条更新记录得到，这个字段定义如下：

```java
`contractBeginTime` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT ''
```
该字段是在记录产生的时候默认赋值为'0000-00-00 00:00:00'，通过jpa获取这条记录的时候，我发现jpa是读取不到该字段的值的，也就是说**该字段映射到java对象中值为null**，那么之前奇怪的现象就
恍然大悟了，jpa在**flush update**的时候该字段更新为null，而数据库**DDL定义**该字段是**非空**的，所以造成了该问题的出现。

**解决方案** 因为该字段只是一个签订日期的含义，所以直接赋值为java中的元时间。


**当jpa使用默认自带封装持久层查询方法**

整个服务方法大致是这样写的
```java
// 步骤1，自带封装findById方法
select status from user where address = 111; // 此时status为1

// 步骤2，自定义修改方法
update user set status = 0 where user_id = 1;// 此时status为0

// 步骤3，自带封装findById方法
select status from user where user_id = 1; // 此时数据库中status为0，但是查出来为1
```

对，没错，它的机制就是这样，步骤1从数据库拿数据后，步骤3他居然从entitymanage里面的sessioncache取值，而不是数据库...

![此处输入图片的描述](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2019-10-18-inside-the-pit-record/4.png)

**解决方案** 在继承JpaRepository的dao方法，也就是上述步骤2的update方法的modifying注解使用**@Modifying(clearAutomatically = true)**


**e.printStackTrace()导致堆栈溢出**

e.printStackTrace() 方法打印堆栈信息到控制台，如果异常处理区域设置过大且堆栈信息过多，加上该请求频繁，会造成字符串常量池很快就满了，多线程会造成死锁，进而导致服务器的奔溃