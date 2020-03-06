---
layout: post
title: 'java序列化'
subtitle: 'java序列化'
date: 2018-05-08
categories: java
author: yates
cover: 'http://cctv.com'
tags: java
---

### 什么是序列化

Java序列化技术正是将**对象转变成一串由二进制字节组成的数组**，可以通过将二进制数据保存到磁盘或者传输网络，磁盘或者网络接收者可以在对象的属类的模板上来反序列化类的对象，达到对象持久化的目的。

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

- 序列化对象里面的**属性**是对象的话也要**实现序列化**接口。

- 类的对象序列化后，类的序列化ID不能轻易修改，不然反序列化会失败。

- 类的对象序列化后，类的**属性**有**增加或删除**不会影响反序列化，只是**值会丢失**。

- 如果父类序列化了，**子类**会**继承**父类的**序列化**，子类无需添加序列化接口。

- 如果**父类没有序列化**，子类序列化了，子类中的属性能正常序列化，但**父类的属性会丢失**，不能序列化。

- 用**Java序列化**的二进制字节数据只能由**Java反序列化**，不能被其他语言反序列化。如果要进行前后端或者不同语言之间的交互一般需要将对象转变成Json/Xml通用格式的数据，再恢复原来的对象。

- 如果某个字段不想序列化，在该字段前加上transient关键字即可

### transient关键字

- 一旦变量被**transient修饰**，变量将**不再是对象持久化**的一部分，该变量内容在序列化后无法获得访问。

- transient关键字**只能修饰变量**，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

- 被transient关键字修饰的变量不再能被序列化，一个**静态变量**不管是否被transient修饰，**均不能被序列化**。

- 对象的序列化可以通过实现两种接口来实现，若**实现Serializable接口**，则所有的序列化将会自动进行，若**实现Externalizable接口**，则没有任何东西可以自动序列化，需要在**writeExternal方法中进行手工指定**所要序列化的变量，这与是否被transient修饰无关。

### Java序列化的缺陷

大多数RPC框架很少使用JDK提供的Java序列化，

- **无法跨语言**：如果基于不同于以编写的应用程序相互通信，则服务实现两个应用服务之间传输对象序列化和反序列化
- 易被攻击：java安全编码方针中说明，对**不信任数据进行反序列化**，是很危险的。对于**长时间进行反序列化对象**，会导致**hashCode方法调用**次数呈次方爆发式增长，从而引发**栈溢出**异常。
- 序列化后流太大：Java序列化使用**objectOutputStream**实现对象转二进制比**byteBuffer**实现的二进制编码数组都**大几倍**。
- 序列化性能太差：和上面一样，序列化**速度比ByteBuffer慢几倍**

**Protobuf**
Protobuf在总多序列化框架中，无论编码，解码，速度，二进制流压缩大小都名列前茅

Protobuf采用T-L-V数据格式，

- T表示正数序列，代表对象每个字段
- L表示Value的字节长度
- V表示字段值经过编码后的值


![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/performance/4.png)