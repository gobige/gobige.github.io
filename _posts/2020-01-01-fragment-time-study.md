﻿---
layout: post
title: '碎片化随笔'
subtitle: '碎片化学习笔记'
date: 2020-01-01
categories: 随笔
author: yates
cover: 'www.baidu.com'
tags: 随笔
---

- 单库-分组（线性提升数据库读性能，消除读写锁，提升写性能；读高可用）
- 分片（分表依然公用一个数据库文件，仍有磁盘IO竞争；分库容易数据迁移不同数据库实例，扩展好）（线性提升数据库写能力，降低单库容量）
- 垂直切分（降低单库容量，降低磁盘IO，跟业务紧密相关）

使用spring-session 和 redis 实现 session集群共享

for循环改写 for(int i = 0; int temp = xxx.size();i < temp;i++)

引起FULL GC原因，老年代不够用了，永久代元空间不够用了，system.gc
 

jdk动态代理是基于接口代理，而CGLIB则不是
spring 5.x 中AOP默认使用JDK动态代理

springboot 2.x默认使用CGLIB，要想使用JDK 配置项  spring.aop.proxy-target-class=false
 
除了构造注入会导致强依赖以外，一个bean也会强依赖于暴露他的配置类。不要对有@configuration注解的配置类进行field级的依赖注入

hikaricp连接池的使用，spring 事务利用AOP原理进行round的方法增强，增强方法调用内部方法时，必须通过引用XXservice.method方法调用才会是事务生效，否则不生效

幂等

1. 查询操作
2. 删除操作
3. 唯一索引，防止新增脏数据
4. token机制
防止页面重提交：
集群环境：采用token加redis；
单机环境token加jvm内存；
处理流程： 1.数据提交前服务器申请token，token放到redis或内存，设置token有效时间 2提交后验证token并删除，表示验证成功并生成新token返回；注意：select+update很容易造成并发问题。应直接采用delete操作并判断执行结果

5. 悲观锁
6. 乐观锁
	- 版本号
	- 条件限制
	- 分布式锁
	- select insert：并发不高的环境，如管理员后台等操作使用
	- 状态机：比如任务，审核流程流转等，上一个状态无法在下一个状态之后发生等
	- 对外提供接口，如付款，通过提交请求的附带 source来源，seq序列号决定唯一性
	

hikaricp连接池的使用，spring 事务利用AOP原理进行round的方法增强，增强方法调用内部方法时，必须通过引用XXservice.method方法调用才会是事务生效，否则不生效
 
reactive programming：响应式编程，面向数据流和变化传播的编程方式

Flux-onNext() onComplete() onError() 

mybatis和jpa的选择：
jpa

jdbctemplate jparespository,mongotemplate mongoresipository,redistemplate redisresipository,redis哨兵和集群

Reactor:一个轻量级JVM基础库，帮助你的服务或应用高效，异步地传递消息

- 高效：消息从A到B产生很少的内存垃圾 2解决消费者处理消息效率低于生产者带来的溢出问题 3尽可能提供非阻塞异步流

reactive:面向数据流和变化传播的编程范式


**常见排序问题**

- 不加order by排序时，oracle查询的数据是混乱的，mysql由于聚餐索引，数据是有序的
- oracle的Null和Null值是无法比较的，既不是相等也不是不相等。对应空字符串，MySql是字符串长度为0空串，Oracle对其作为Null处理


**arthas线上诊断工具使用**

**热部署**

平常碰到一些线上bug，为了定位问题，和解决问题，需要修改源代码

- 反编译class文件：jad --source-only com.yates.project.TestService > /tmp/TestService.java
- 修改java文件：vim /tmp/TestService.java
- 查询源文件classLoader：sc -d com.yates.project.TestService | grep classLoaderHash
- 编译源文件：mc -c asd2ff1 /tmp/TestService.java -d /tmp
- 热更新：redefine /tmp/SchedulingTaskController.class

注意：很多时候使用MC编译时会报错，所以建议使用还能Idea编译后再使用redefine更新class文件

arthas其他指令：

- dashboard：查看线程情况，内存使用情况
- thread：当前线程信息，查看线程的堆栈
- jvm：查看jvm情况
- vmoption：查看jvm配置参数
- logger:查看日志配置 logger --name ROOT --level DEBUG:更新日志级别
- heapdump：dump堆栈信息
- monitor：查看某个方法的调用情况,调用次数，成功次数，RT
- watch：方法执行情况，入参，返回值，异常
- trace：方法链路追踪，性能开销，调用链路
- tt:记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

**Class.forName和Classloader区别**
前者会自动初始化类（JDBC），后者不会（Spring Ioc），



**可执行jar包**
Jar描述，META-INF/MAINFEST.MF 描述main-class：JarLauncher  执行 start-class
Spring Boot Loader 
项目内容
项目依赖

配置HTTP2需要java9，tomcat9 以上版本，配置SSL，配置属性项 server.http2.enabled。
HTTP库中OkHttp支持HTTP2，RestTemplate配置 OkHttp3ClientHTTPRequestFactory

web容器配置也可以通过springboot属性配置，编程方式也可以实现

**12-Factor-App**
基准代码：一份基准代码，多份部署
依赖：显示声明依赖关系
配置：环境中存储配置
后端服务：把后端服务当做资源
构建，发布，运行：严格分离构建与运行
进程：以一个或多个无状态进程运行应用
端口绑定：通过端口绑定提供服务
并发：通过进程模型进行扩展
易处理：快速启动和优雅终止可最大化健壮性
开发环境与线上环境等价：尽可能保持开发，预发布，线上环境相同
日志：把日志当做事件流
管理进程：后台管理任务当做一次性进程运行


Eureka，zookeepeer，consul，nacos都可作为服务注册中心
@EnableEurekaServer  允许作为服务注册
@EnableFeignClients  允许使用feign调用服务，

@LoadBalanced

@EnableEurekaClient  允许服务发现，调用

@EnableCircuitBreaker 允许发生服务熔断

feign.hystrix.enabled=true Feign支持服务熔断

@EnableHystrixDashboard观察熔断情况

@HystrixCommand(fallbackMethod = "fallback") 客户顿调用服务发生熔断机制
* 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
* 请求总数下限：在快照时间窗内，必须满足请求总数下限才有资格根据熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用此时不足20次，即时所有的请求都超时或其他原因失败，断路器都不会打开。
* 错误百分比下限：当请求总数在快照时间窗内超过了下限，比如发生了30次调用，如果在这30次调用中，有16次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%下限情况下，这时候就会将断路器打开。

hystrix停止维护
resilience4j，一款轻量级抑郁使用容错库，可以提供熔断，限流的功能

spring cloud stream：一款构建消息驱动的微服务应用轻量级框架，发布订阅，消费组，分区；声明式编程；支持mq，kafka等中间件。
通过与Binder直接打交道，应用生产者，消费者与消费系统之间桥梁

spring cloud Sleuth：服务链路治理：
zipkin 支持web和mq

服务治理：架构设计是否合理，哪些链路时关键链路，链路容量水位趋势，系统变更管理和审计，微观上，系统依赖了什么，有哪些配置，主观与客观质量


 properties:
      hibernate:
        format_sql: true
        use_sql_comments: true

        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>

使用springcloud作为微服务，在一个活动结束返还库存的任务中，因为商品很多，造成服务调用发生超时行为，hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 10000
feign.client.config.default.readTimeout: 10000
ribbon.ReadTimeout: 10000

设置服务调用超时等待时间
 
1 定义用户真正的需求目的
2 结合基本操作解决问题，