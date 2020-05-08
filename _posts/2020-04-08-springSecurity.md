---
layout: post
title: 'SpringSecurity认证源码分析'
subtitle: 'SpringSecurity'
date: 2020-04-08
categories: Spring
author: yates
cover: 'www.baidu.com'
tags: Spring
---

## Zuul 请求处理流程

1. 请求到达ZuulServlet（继承 HttpServlet），初始化ZuulRunner，RequestContext 填充request和response。   
2. 调用 ZuulRunner分别pre，route，post类型的filter
3. 通过FilterLoader 获取对应filter type的  filter集合
	- FilterRegistry返回 ConcurrentHashMap类型的 filters 集合
		- 通过spring IOC 把注入的所有 filter，通过 ZuulFilterConfiguration配置 初始化到 ZuulFilterInitializer 中
		- ZuulFilterInitializer 对ConcurrentHashMap类型的类型的 filterRegistry对象进行数据put
		- 把filterRegistry对象 按照 filterType分类，赋值为 ConcurrentHashMap<String, List<ZuulFilter>> 类型的 hashFiltersByType对象
4. 通过 FilterProcessor对象 处理对应filter类型的 filter集合
	- 得到ZuulFilter对象，调用对象本身runFilter方法
	
	
## oauth/token接口请求流程

1. 调用 TokenEndpoint.postAccessToken方法进行认证，授权
	- 前端传入 grant_type&client_id&client_secret&scope等参数
	- 通过ClientDetailsService接口根据clientId获取 ClientDetails对象
	- 通过DefaultOAuth2RequestFactory根据 scope，grant_type，ClientDetails等参数 创建TokenRequest对象
2. 通过继承AbstractTokenGranter自定义类，调用getAccessToken方法获取token
	- 根据重写getOAuth2Authentication方法进行**认证**，
	- 根据 DefaultTokenServices 创建返回 OAuth2AccessToken对象
		- 通过 TokenStore接口实现持久化方式类获得token
			- （JWT方式）生成一个过期时间为30天， 的refresh token对象
			- 生成一个过期时间为12小时的 Basic access token对象
			- 通过TokenEnhancer接口实现类对Basic access token对象进行增强（token加密）
		- 通过TokenStore接口实现类 存储accessToken和refreshToken
3. 返回tokenEntity类

通过在应用接口层 实现AuthenticationEntryPoint接口

通过配置的security.oauth2.resource.user-info-uri 参数每次调用一次后台接口都会去passport-server请求一次 **oauth/user接口获取用户session信息

 org.springframework.security.access.AccessDeniedException: Access is denied