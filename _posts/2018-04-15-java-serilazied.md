---
layout: post
title: 'java序列化'
subtitle: 'java序列化'
date: 2018-03-15
categories: java
author: yates
cover: 'http://cctv.com'
tags: java
---

### 什么是序列化

Java序列化技术正是将对象转变成一串由二进制字节组成的数组，可以通过将二进制数据保存到磁盘或者传输网络，磁盘或者网络接收者可以在对象的属类的模板上来反序列化类的对象，达到对象持久化的目的。

### 2、序列化/反序列化

可以借助springframework工具包里面的类实现对象的序列化及反序列化，你没有必要自己写。

```java
 public static void main(String[] args) {
       Apple apple = new Apple(1,2,"china");
        byte[] bytes = SerializationUtils.serialize(apple);
        Apple a = (Apple)SerializationUtils.deserialize(bytes);
        System.out.println(a.toString());
    }
```

### 序列化注意事项

- 序列化对象必须实现序列化接口。

- 序列化对象里面的属性是对象的话也要实现序列化接口。

- 类的对象序列化后，类的序列化ID不能轻易修改，不然反序列化会失败。

- 类的对象序列化后，类的属性有增加或者删除不会影响序列化，只是值会丢失。

- 如果父类序列化了，子类会继承父类的序列化，子类无需添加序列化接口。

- 如果父类没有序列化，子类序列化了，子类中的属性能正常序列化，但父类的属性会丢失，不能序列化。

- 用Java序列化的二进制字节数据只能由Java反序列化，不能被其他语言反序列化。如果要进行前后端或者不同语言之间的交互一般需要将对象转变成Json/Xml通用格式的数据，再恢复原来的对象。

- 如果某个字段不想序列化，在该字段前加上transient关键字即可