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


### 注解
@Controller：修饰class，用来创建处理http请求的对象
@RestController：@Controller和@ResponseBody的结合体，使其接口返回json格式。
@RequestMapping：url映射地址