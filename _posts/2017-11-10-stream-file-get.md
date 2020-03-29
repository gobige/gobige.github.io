---
layout: post
title: '获取配置文件的方式'
subtitle: '获取配置文件的流'
date: 2017-11-10
categories: IO
author: yates
cover: 'http://cctv.com'
tags: IO
---

### 获取配置文件


1. 通过类加载器获取配置文件
```java
this.getClass().getResourceAsStream(path)
```

```java
     public InputStream getResourceAsStream(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResourceAsStream(name);
        }
        return cl.getResourceAsStream(name);
    }
```
先通过resolveName解析为相对路径，然后通过getClassLoader从子类开始向上查找类加载器
 
2. 通过java提供的文件流方式
```java
new FileInputStream(new File(filePath));
```

3. 通过URL来获取
```java
InputStream stream = new URL("path").openStream();
```
  

4. 在servlet中，还可以通过context来获取InputStream
```java
InputStream in = context.getResourceAsStream("filePath");
```
2. 加载stream流方式
在java中经常把流数据加载封装到properties结构中来进行对象存储

```java
Properties prop = new Properties();
prop.load(inputStream);
```

java jdk提供 方法直接获取对象也是间接使用properties

```java
ResourceBundle resource = new PropertyResourceBundle(fileInputStream);
String resourcekey = resource.getString("test.name");	

ResourceBundle resourceBundle = configFileReadTest.getResourceBundle("test");
String Bundlekey = resourceBundle.getString("test.name");
```