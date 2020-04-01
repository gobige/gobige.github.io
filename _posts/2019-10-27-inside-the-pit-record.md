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

# 记录一些实践中踩过的坑
 
## 资源文件在java工程中的目录问题
这个问题是一老友问的我。情况是这样的，我们一般的java工程目录都是分为java，test,resource目录，一般资源，配置文件都是放在resource下，老友把**资源文件放在java目录**和
和java源文件放在一起，通过**代码读取配置文件中的数据，怎么也读取不到**，于是问我为什么，我看了下这个问题，然后自己试了试，确实是不行的。想了想，然后在target目录下把编译好的class
目录打开，发现并没有找到该资源文件，于是这个问题就明了了，java编译过程中只会识别java后缀的文件进行**编译**，而在之后的**类加载过程**中也只会读取class后缀的文件，所以自然在java目录下存放资源文件就不能被读取了。
而存放于resource目录下的文件会直接作为resource root移到target包下

## jpa之你真的了解jpa的执行机制吗？

### **当jpa自定义sql遇上数据库某非空dateTime类型默认为'0000-00-00 00:00:00'的数据**

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


### **当jpa使用默认自带封装持久层查询方法**

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


### **e.printStackTrace()导致堆栈溢出**

e.printStackTrace() 方法打印堆栈信息到控制台，如果异常处理区域设置过大且堆栈信息过多，加上该请求频繁，会造成字符串常量池很快就满了，多线程会造成死锁，进而导致服务器的奔溃

### **数据库建表**
像三方接口得到的数据随时可能会变的，不宜在该表中添加一些**自定义**字段，容易造成以后调用接口就会考虑其是不是应该赋值

表结构中禁止**one，two，three**等字段，应该使用一对多的表存储

商品**审核**这种，因为涉及的比较多，对于增加和修改商品如果使用**拷贝**一份主表的数据记录拿出**修改**，然后审核通过后覆盖现有的记录，会导致问题
1. 表关联比较复杂
2. 供应商添加的商品需要看到（即使还在审核中），基于商品表数量不像订单表一样大数据量，采用修改，增加内容直接在主表操作，通过增加**额外的标识字段**来区别，如果通过，就是用**修改的数据覆盖现有数据**

面对关系很复杂的表关联设计时,使用中间关联表,关联各表之间的联系

### **分页查询**
**为什么mysql分页会出现重复数据**

- mysql优化器在遇到order bylimit语句时，使用**priority queue**，在不能使用索引有序性时，如果要排序并limit N时，只需在排序过程中保留N条记录即可。而**5.6版本priority queue**使用了堆排序。**堆排序时不稳定**的。

**分页是建立在排序基础上，进行数量的范围的分隔。排序是数据库提供的功能，分页却是衍生出来的应用需求**

### **Mybatis 返回主键ID**
当要使用 **useGeneratedKeys** 返回生成记录的主键时，下列的dao接口映射，是返回不了主键id的
`void insertOssDepartment(@Param("item")OssDepartment item);`

正确做法是
`void insertOssDepartment(OssDepartment item);`

### **Mybatis注解**
如果使用mybatis中的CRUD注解@select,在没有xml配置映射的情况下使用的是**全局驼峰映射**,那么domain对象的属性对应数据库字段一定要规范;或者使用@Results @Result 注解填写映射关系



### **Spring**
springboot2.5.8升级时默认Integer和date是不能互相转换，会报错，而之前版本时可以的

**BeanUtils的坑**
工具用不好，就是坑自己。

1. Spring的BeanUtils的CopyProperties方法需要对应属性有get和set方法
2. 两个变量类型不一样，不会赋值
3. 是一种浅度克隆，赋值的如果是引用数据类型，不会重新生成对象


### Spring事务
@cacheevict方法和@transactional事务同时用的情况下很可能导致redis刷新缓存失败，参考aop round就像一层一层的包裹与解开必然执行顺序和提交顺序会有差别，解决方案：缩小事务范围

spring 事务利用AOP原理进行round的方法增强，增强方法调用内部方法时，必须通过引用XXservice.method方法调用才会是事务生效，否则不生效

事务传播行为：

- PROPAGATION_REQUIRES_NEW：始终启动一个新事务，两个事务无关联
- PROPAGATION_REQUIRED_NESTED：在原事务内启动一个内嵌事务，两个事务有关联，外部事务回滚，内部也会回滚


### **Java API 集合List**
集合中获取迭代器iterator时必须加上**泛型**声明类，不然迭代是无法进行**删除等操作**

### **Java Stream操作**
collector.toMap，key,value不能为空，key，value不能重复(k,v)->v  支持map集合插入重复数据时覆盖原有数据

### **Java Time**
LocalDateTime.parse只能解析年月日时分秒格式的字符串,不能解析单独只有年月日的格式

### **中间件 RabbitMQ**

RabbitMQ 消息类不能传递 date类型参数

RabbitMQ 消息类必须有默认构造函数

mq的编写 一定要分清什么事次要的可以放后的,而不是所有的东西都适用mq

RabbitMQ在使用的时候，因为有重试的机制。那么在消费者调用其他模块服务的时，服务一定要可靠（简单，稳定，不会有太多复杂逻辑），而且带事务回滚，如果失败会不断重试，如果没有回滚会导致许多脏数据，如果调用无法回滚的如：三方接口，那么一定要在mq里面进行异常处理，避免不断重试

### **GIT**
git rebase 是一个危险命令，因为它**改变了历史，我们应该谨慎使用。除非你可以肯定该feature分支只有你自己使用，否则请谨慎操作。结论：只要你的分支上需要 rebase 的所有 commits 历史还没有被 push 过，就可以安全地使用 git-rebase来操作。

### **Maven**
idea java文件显示J不显示C,用maven引入方式

### 业务
页面配置问题：很多需要先删除页面元素，再增加新的页面元素逻辑，在高并发的情况下，导致删除不到应该删除的数据

1. 前端按钮失效控制  
2. 后端接口请求频率控制
3. 修改元素获取方式，通过页面为维度，索引获取数据

类似于一张表的操作一定要搞明白每一个字段的含义，特别多个流程设计修改一张表数据的时候，要考虑各种各样的情况。

但是不建议一张表代表的含义过多，比如库价，库存表，普通商品库存操作会修改，活动商品操作也会修改，指不定哪个流程就就互相影响了，所以表能独立最好还是独立

考虑性能情况下，需要融合到一张表
1. 一定要搞清楚这张表每个字段含义，
2. 已经涉及表修改的所有功能业务熟系，以便去写相应的处理逻辑。