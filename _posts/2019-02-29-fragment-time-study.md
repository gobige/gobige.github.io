---
layout: post
title: '碎片化随笔'
subtitle: '碎片化学习笔记'
date: 2019-02-29
categories: 随笔
author: yates
cover: 'www.baidu.com'
tags: 随笔
---


Spring Bean 加载顺序：Constructor >> @Autowired >> @PostConstruct

在普通DDL执行语句后面加上 ,ALGORITHM=INPLACE, LOCK=NONE; 使用onlineDDL，5.6版本以上才有

String字符串和char字符数组：String字符串是不可变的；内存中存在直到被GC清理，char字符数组可以通过设置为空白和零。所以在密码存储方面场景来说字符数组更合适，不过还是应该加密

timestamp 记录的时间，时区自动处理，能够支持时间范围：1000-9999
datetime 记录固定时间，不会变化，能够支持时间范围：1970-2038


mysql数据库时间类型 有datetime和timestamp

- 设置默认时间设置： DEFAULT CURRENT_TIMESTAMP 
- 设置时间自动更新： ON UPDATE CURRENT_TIMESTAMP
 

logging.level.com.carhouse.setting.core: DEBUG 打印sql日志
	
server.servlet.context-path：springmvc 添加 默认请求路由 前缀

instanceof    一个对象实例是否是一个类或接口 或 其子类子接口  的实例。   

isAssignableFrom   类Class1和 类Class2是否相同 或 类Class1 是 类Class2 的超类或接口。   
通常调用格式是  

代码中使用 logger.isDebugEnabled()判断，有效优化 字符串拼接使用时的开销
 
软件的本质复杂性存在于复杂的业务需求中。没有架构，所有代码耦合在一起，人类的心智模型不擅长处理复杂性。架构出现时为了管理复杂性，以取得更高的生产力。

不管是MVC还是MVP,MVVM，都存在一个问题，没有处理好业务流程，界面转场。
 
JPA Level 2 Cache：共享缓存

实体Entity使用@Cacheable注解开启二级缓存

SharedCacheMode 

- NONE：关闭二级缓存
- ALL：对所有entity应用二级缓存
- DISABLE_SELECTIVE：对所有@Cacheable(true)注解实体适用
- ENABLE_SELECTIVE：对所有@Cacheable(false)注解实体适用
- UNSPECIFIED：使用默认

失效缓存

```java
Cache cache = entityManagerFactory.getCache();
cache.evict(Student.class, 1);
```

缓存存储模式 CacheStoreMode

 
- BYPASS：读取，提交的时候不增加和不更新到缓存
- REFRESH：默认
- USE.：读取，提交的时候更新，增加到缓存
 
    
缓存读取模式   CacheRetrieveMode

- BYPASS：不从缓存读取
- USE.：从缓存读取

```java
Query query = entitymanager.createQuery("select student FROM Student student");
query.setHint("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
```       


properties:
  javax:
	persistence:
	  cache:
		retrieveMode: javax.persistence.CacheRetrieveMode.BYPASS
		
- 分布式事务处理，例如转账，A账户校验，金额扣减，生成待处理流水记录-》调用B账户MQ消息生成，后台线程执行三方调用，更新流水记录-》返回
mq消息丢失处理：后台定时任务补偿扫描超时待处理流水记录放入MQ消息
重复消费处理：消费者第一次消费生成消费流水记录，再次消费时判断请求流水号是否重复，保证幂等性


- pwdx [pid]:根据pid查找对应业务进程路径

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
	

hikaricp连接池的使用

spring 事务利用AOP原理进行round的方法增强，增强方法调用内部方法时，必须通过引用XXservice.method方法调用才会是事务生效，否则不生效
 
reactive programming：响应式编程，面向数据流和变化传播的编程方式

Flux-onNext() onComplete() onError() 

mybatis和jpa的选择

jdbctemplate jparespository,mongotemplate mongoresipository,redistemplate redisresipository,redis哨兵和集群

Reactor:一个轻量级JVM基础库，帮助你的服务或应用高效，异步地传递消息

- 高效：消息从A到B产生很少的内存垃圾 2解决消费者处理消息效率低于生产者带来的溢出问题 3尽可能提供非阻塞异步流


**常见排序问题**

- 不加order by排序时，oracle查询的数据是混乱的，mysql由于聚簇索引，数据是有序的
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
	- watch com.carhouse.product.core.service.impl.GoodsSearchResult4BServiceImpl selectGoodsList "{params,returnObj}" -x 2
- trace：方法链路追踪，性能开销，调用链路
	- trace com.carhouse.api.controller.v3.SupplierStoreController index
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

- 基准代码：一份基准代码，多份部署
- 依赖：显示声明依赖关系
- 配置：环境中存储配置
- 后端服务：把后端服务当做资源
- 构建，发布，运行：严格分离构建与运行
- 进程：以一个或多个无状态进程运行应用
- 端口绑定：通过端口绑定提供服务
- 并发：通过进程模型进行扩展
- 易处理：快速启动和优雅终止可最大化健壮性
- 开发环境与线上环境等价：尽可能保持开发，预发布，线上环境相同
- 日志：把日志当做事件流
- 管理进程：后台管理任务当做一次性进程运行


Eureka，zookeepeer，consul，nacos都可作为服务注册中心

- @EnableEurekaServer  允许作为服务注册
- @EnableFeignClients  允许使用feign调用服务，
- @LoadBalanced
- @EnableEurekaClient  允许服务发现，调用
- @EnableCircuitBreaker 允许发生服务熔断
- feign.hystrix.enabled=true Feign支持服务熔断
- @EnableHystrixDashboard观察熔断情况
- @HystrixCommand(fallbackMethod = "fallback") 客户端调用服务发生熔断机制条件

** 快照时间窗**：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。

** 请求总数下限**：在快照时间窗内，必须满足请求总数下限才有资格根据熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用此时不足20次，即时所有的请求都超时或其他原因失败，断路器都不会打开。

** 错误百分比下限**：当请求总数在快照时间窗内超过了下限，比如发生了30次调用，如果在这30次调用中，有16次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%下限情况下，这时候就会将断路器打开。

hystrix停止维护，resilience4j，spring官方背书，一款轻量级易于使用容错库，可以提供熔断，限流的功能

spring cloud stream：一款构建消息驱动的微服务应用轻量级框架，发布订阅，消费组，分区；声明式编程；支持mq，kafka等中间件。
通过与Binder直接打交道，应用生产者，消费者与消费系统之间桥梁

spring cloud Sleuth：服务链路治理：
zipkin 支持web和mq

服务治理：架构设计是否合理，哪些链路时关键链路，链路容量水位趋势，系统变更管理和审计，微观上，系统依赖了什么，有哪些配置，主观与客观质量

JPA打印sql
```java
properties:
      hibernate:
        format_sql: true
        use_sql_comments: true
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
```

springcloud服务调用超时

在一个活动结束返还库存的任务中，因为商品很多，造成服务调用发生超时行为，hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 10000
feign.client.config.default.readTimeout: 10000
ribbon.ReadTimeout: 10000
 
1. 定义用户真正的需求目的
2. 结合基本操作解决问题，
