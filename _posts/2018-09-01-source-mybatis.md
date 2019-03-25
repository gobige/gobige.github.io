---
layout: post
title: '源码解析-mybatis'
subtitle: 'mybatis的源码解析'
date: 2018-09-01
categories: 源码
author: yates
cover: ''
tags: source
---

**mybatis调用实例**
```java
public static void main(String[] args) {
        String resource="mybatis-config.xml";
        InputStream inputStream = null;
        try {
            inputStream = Resources.getResourceAsStream(resource);  1
        } catch (IOException e) {
            e.printStackTrace();
        }
        SqlSessionFactory sqlSessionFactory=null;
        sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream); 2
        SqlSession sqlSession=null;
        try {
            sqlSession=sqlSessionFactory.openSession(); 3
            SysAppVersionInfoDao sysAppVersionInfoDao = sqlSession.getMapper(SysAppVersionInfoDao.class); 4
            SysAppVersionInfo role = sysAppVersionInfoDao.queryByIdBySelectProvider(16);  5
            System.out.println(role.getId()+":"+role.getRemark()+":"+role.getAppVersion());
            sqlSession.commit(); 6 

        } catch (Exception e) {
            sqlSession.rollback(); 6
            e.printStackTrace();
        }finally {
            sqlSession.close();  7
        }
    }
```

mybatis调用大体可分为上面7步：

1. 加载解析mybatis的xml配置文件
2. 根据得到的配置文件流建立sqlsesiionfactory
3. 通过sqlsessionfactory为此次sql执行创建sqlsession
4. 通过反射为sqlsession加载对应的mapper映射
5. 通过sqlsession执行对应的sql语句
6. 提交/回滚sql执行语句
7. 关闭sqlsession，释放资源


**详解**

第一步：
此步的解析请移步
http://muyibeyond.cn/2017/11/11/stream-file-get.html

第二步：
通过得到的流装载封装到XPathParser对象中，使用XMLConfigBuilder对XPathParser解析（XPathParser把流数据转换成XNODE，从configuration根元素开始，通过对一个个节点的eval得到各个node封装到XNODE对象，属性，变量，XPATH）

XNODE对象 声明的变量
```java
private final Node node;
private final String name;
private final String body;
private final Properties attributes;
private final Properties variables;
private final XPathParser xpathParser;
```

XPathParser对象 声明的变量
```java
private final Document document;
private boolean validation;
private EntityResolver entityResolver;
private Properties variables;
private XPath xpath;
```

XPathParser对象 声明的变量
```java
private boolean parsed;
private final XPathParser parser;
private String environment;
private final ReflectorFactory localReflectorFactory;
```

xmlconfigbuilder解析
```java
public Configuration parse() {
    if (this.parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    } else {
        this.parsed = true;
        this.parseConfiguration(this.parser.evalNode("/configuration"));
        return this.configuration;
    }
}
```

xmlconfigbuilder解析 XNode
```java
private void parseConfiguration(XNode root) {
    try {
        this.propertiesElement(root.evalNode("properties"));
        Properties settings = this.settingsAsProperties(root.evalNode("settings"));
        this.loadCustomVfs(settings);
        this.typeAliasesElement(root.evalNode("typeAliases"));
        this.pluginElement(root.evalNode("plugins"));
        this.objectFactoryElement(root.evalNode("objectFactory"));
        this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
        this.settingsElement(settings);
        this.environmentsElement(root.evalNode("environments"));
        this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        this.typeHandlerElement(root.evalNode("typeHandlers"));
        this.mapperElement(root.evalNode("mappers"));
    } catch (Exception var3) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
    }
}
```