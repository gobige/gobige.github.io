---
layout: post
title: 'dubbo一些好用的功能'
subtitle: 'dubbo一些好用的功能'
date: 2018-02-12
categories: dubbo
author: yates
cover: ''
tags: dubbo
---
 
#### 直连provider
通过设置消费者配置文件中连接服务的url属性来调用指定机器的服务,主要用于开发环境,本地调试
```java
<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" version="1.0.0"  url="dubbo://172.18.1.205:20888/" />
```

#### 多版本

当一个接口实现,出现不兼容升级时,可以使用版本号过渡,版本号不同的服务相互间不引用
```java
<dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" version="1.0.0" />
```
比如两个相同服务一个version为1.0.0一个版本为1.0.1,消费者使用version='*',那么两个服务共同承担一半的流量,可以实现不停机平滑的升级

#### 回声测试
用于检测服务是否可用,常用于监控.使用方式:所有服务自动实现echoservice接口,只需将任意服务引用做强制转换
```java
EchoService echoService = (EchoService) demoService;
System.out.println(echoService.$echo("hello"));
```

#### 隐式参数
比如conttroller层接受token的一个参数,然后通过RpcContext在服务消费者和服务提供者之间传递参数(1多个service级联调用可多级传递,2只能传递string类型数据)
```java
RpcContext.getContext().setAttachment("CRT_MEMBER_ID", "13828886888");
 
RpcContext.getContext().getAttachment("CRT_MEMBER_ID")
```

#### 本地伪装
用于服务降级,当某个服务挂掉后,客户端通过mock数据返回授权失败,而不是抛出异常
```java
<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService"                 version="1.0.0" mock="com.alibaba.dubbo.demo.consumer.mock.DemoServiceMock"/>
```


#### 访问日志
如果想记录每次请求消息,可开启访问日志,此日志量比较大,请注意磁盘容量
```java
// 全局配置
<dubbo:provider accesslog="/app/dubbo-demo.log"/>

// 局部配置
<dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" accesslog="/app/demo.log"/>
```

#### 延迟暴露
如果服务需要预热,比如初始化本地缓存等,可以是哟功能delay进行言辞暴露,使dubbo在spring容器初始化完成后延迟多少毫秒在暴露服务
```java
// 全局配置
<dubbo:provider delay="5000"/>

// 局部配置
<dubbo:service delay="5000" interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" version="1.0.0"/>
```


请求线程的 applicationContext.getBean() 调用，先同步 singletonObjects 判断 Bean 是否存在，不存在就同步 beanDefinitionMap 进行初始化，并再次同步 singletonObjects 写入 Bean 实例缓存。
而 Spring 初始化线程，因不需要判断 Bean 的存在，直接同步 beanDefinitionMap 进行初始化，并同步 singletonObjects 写入 Bean 实例缓存。