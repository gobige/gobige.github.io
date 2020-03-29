---
layout: post
title: '��ȡ�����ļ��ķ�ʽ'
subtitle: '��ȡ�����ļ�����'
date: 2017-11-10
categories: IO
author: yates
cover: 'http://cctv.com'
tags: IO
---

### ��ȡ�����ļ�


1. ͨ�����������ȡ�����ļ�
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
��ͨ��resolveName����Ϊ���·����Ȼ��ͨ��getClassLoader�����࿪ʼ���ϲ����������
 
2. ͨ��java�ṩ���ļ�����ʽ
```java
new FileInputStream(new File(filePath));
```

3. ͨ��URL����ȡ
```java
InputStream stream = new URL("path").openStream();
```
  

4. ��servlet�У�������ͨ��context����ȡInputStream
```java
InputStream in = context.getResourceAsStream("filePath");
```
2. ����stream����ʽ
��java�о����������ݼ��ط�װ��properties�ṹ�������ж���洢

```java
Properties prop = new Properties();
prop.load(inputStream);
```

java jdk�ṩ ����ֱ�ӻ�ȡ����Ҳ�Ǽ��ʹ��properties

```java
ResourceBundle resource = new PropertyResourceBundle(fileInputStream);
String resourcekey = resource.getString("test.name");	

ResourceBundle resourceBundle = configFileReadTest.getResourceBundle("test");
String Bundlekey = resourceBundle.getString("test.name");
```