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
 
 
## Spring IOC 源码分析

当我们运行以下代码时：

```Java
ClassPathXmlApplicationContext applicationContext=new ClassPathXmlApplicationContext("applicationContext.xml");
```
Spring会去加载配置文件，创建环境，创建bean。也就是IOC机制

附上一张依赖图

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-08-26-springsource/2.png)

流程

### **调用构造方法**

```Java
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent); // 合并父容器环境environment
		setConfigLocations(configLocations); // 封装配置文件地址路由为List集合
		if (refresh) { // refresh 加载
			refresh();
		}
	}
```

### **refresh过程**

**prepareRefresh 预处理**

- initPropertySources(no excute)	
- validateRequiredProperties(no excute)
- reset applicationListeners(emplist)
- refreshBeanFactory
	- 生成 DefaultListableBeanFactory
	- loadBeanDefinitions 加载bean到factory
		- 创建XmlBeanDefinitionReader 赋值 resourceLoader， environment，entityResolver
		- loadBeanDefinitions
			- 读取配置文件application.xml流（每读取一个配置文件就加入threadLocal中，避免多线程重复加载），读取完remove
- 配置beafactory，如：ClassLoader和BeanPostProcessor
- invokeBeanFactoryPostProcessors 为bean实例进行扩展处理
	- doGetBeanNamesForType
		- isTypeMatch 匹配给定bean名称是否和beanType匹配。自定义bean和环境bean
- registerBeanPostProcessors
	- primaryOrder，order，noOrder PostProcessor注册
- initMessageSource
	- 查找是否有name为messageSource的bean，如果有放到容器里，如果没有，初始化一个系统默认的放入容器
- initApplicationEventMulticaster
	- 查找是否有applicationEventMulticaster的bean，如果有放到容器里，如果没有，初始化一个系统默认的放入容器
- onRefresh
- registerListeners
	- 注册静态和自定义监听器，并且绑定多播事件
- finishBeanFactoryInitialization
	- 设置 conversionService
	- 注册默认bean后置处理器
	- preInstantiateSingletons 初始化所有非lazy-load的bean
		- 初始化依赖bean对象	
- finishRefresh
	- 清除上下文级别的资源缓存
	- 上下文初始化生命周期处理器。
	- 发布最终事件