---
layout: post
title: 'springboot实战'
subtitle: 'springboot实战'
date: 2018-05-15
categories: 框架
author: yates
cover: 'http://cctv.com'
tags: 框架
---

 
### **Spring MVC**
spring对每个bean提供一个scope属性表示bean的作用域，这些bean大多是无状态对象。single类型的无状态bean都是线程安全的，例如每个dao提供的方法都只是对数据的CURD，每个数据库connection都作为函数局部变量，处于栈中，属于线程私有的内存区域，用完即回收。每个controller被多个线程执行，springmvc对请求拦截粒度时基于每个方法（而struct2时基于类，也就是多例controller，频繁创建，回收，影响性能）。spring没有对bean的多线程安全作出任何保证和措施，所以我们在bean的设计中声明任何有状态的实例变量和类变量，如果非得有，则要使用ThreadLocal变量私有化或使用synchronized，lock，cas等实现线程同步方法

### springBoot的结构

- src/main/java 程序业务代码入口
- src/main/resources 工程配置文件入口：
	- application.properties 工程主配置文件
	- /static 工程静态文件入口
- src/test/  工程测试文件入口


### springboot中常用的注解
- @Controller：修饰class，用来创建处理http请求的对象
- @RestController：@Controller和@ResponseBody的结合体，使其接口返回json格式。
- @RequestMapping：url映射地址
- @ResponseBody：数据返回支持json格式
- @Value：读取配置文件配置
- @Component: 作为组件注入spring容器
- @Service：作为服务注入spring容器
- @EnableSwagger2：应用s'wagger文档（swwagger配置类使用）
- @ApiOperation(value = "swwager文档",notes = "swwager文档") 该方法生成swwager文档注释
- @EnableCaching：开启缓存（缓存配置类使用）
- @EnableAsync：开启异步执行
- @Async：该方法异步执行
- @EnableScheduling：开启定时任务
- @Scheduled：方法作为定时任务
- @Configuration：作为配置注入spring容器
- @Aspect：aop注入，@Pointcut：切入点 @Before：环绕前方法事件；@AfterReturning：环绕后方法事件
- @Transactional(isolation = Isolation.DEFAULT,propagation = Propagation.REQUIRED) 设置当前方法作为一个事务处理，参数可设置隔离级别，设置传播行为
- @ControllerAdvice：作为控制器注入spring容器
- @ExceptionHandler：指定异常处理方法


### spring boot 配置文件加载顺序 


#### **项目内配置文件加载顺序**

SpringBoot项目启动会扫描以下位置的application.properties或者application.yml文件作为SpringBoot的默认配置文件，具体的目录位置见下图。

- file:./config/ （ 项目根路径下的config文件夹）
- file:./ （项目根路径）
- classpath:/config/ （类路径下的config文件夹）
- classpath:/ （类路径） 


#### **外部配置加载顺序SpringBoot也可以从以下位置加载配置**

优先级从高到低，高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置 。

- 命令行参数。所有的配置都可以在命令行上进行指定；
- 来自java:comp/env的JNDI属性；
- Java系统属性（System.getProperties()）；
- 操作系统环境变量 ；
- jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
- jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件 再来加载不带profile
- jar包外部的application.properties或application.yml(不带spring.profile)配置文件
- jar包内部的application.properties或application.yml(不带spring.profile)配置文件
- @Configuration注解类上的@PropertySource 