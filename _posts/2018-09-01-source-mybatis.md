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


第二步：
