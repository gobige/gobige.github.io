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

### springBoot的结构

- src/main/java 程序业务代码入口
- src/main/resources 工程配置文件入口：
	- application.properties 工程主配置文件
	- /static 工程静态文件入口
- src/test/  工程测试文件入口


### springboot中常用的注解
@Controller：修饰class，用来创建处理http请求的对象
@RestController：@Controller和@ResponseBody的结合体，使其接口返回json格式。
@RequestMapping：url映射地址
@ResponseBody：数据返回支持json格式
@Value：读取配置文件配置
@Component: 作为组件注入spring容器
@Service：作为服务注入spring容器
@EnableSwagger2：应用s'wagger文档（swwagger配置类使用）
@ApiOperation(value = "swwager文档",notes = "swwager文档") 该方法生成swwager文档注释
@EnableCaching：开启缓存（缓存配置类使用）
@EnableAsync：开启异步执行
@Async：该方法异步执行
@EnableScheduling：开启定时任务
@Scheduled：方法作为定时任务
@Configuration：作为配置注入spring容器
@Aspect：aop注入，@Pointcut：切入点 @Before：环绕前方法事件；@AfterReturning：环绕后方法事件
@Transactional(isolation = Isolation.DEFAULT,propagation = Propagation.REQUIRED) 设置当前方法作为一个事务处理，参数可设置隔离级别，设置传播行为
@ControllerAdvice：作为控制器注入spring容器
@ExceptionHandler：指定异常处理方法