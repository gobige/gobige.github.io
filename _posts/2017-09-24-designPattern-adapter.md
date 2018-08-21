---
layout: post
title: '设计模式之适配器模式'
subtitle: '适配器模式'
date: 2017-09-24
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---


### 结构型模式
#### 适配器模式
适配器模式是作为两个不兼容的接口之间的桥梁,它结合了两个独立接口的功能。这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能

**优点**

- 可以让任何两个没有关联的类一起运行。 
- 提高了类的复用。
- 增加了类的透明度。 
- 灵活性好。

**缺点**

- 过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了b接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。
- 由于JAVA至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。

**使用场景** 

- 有动机地修改一个正常运行的系统的接口，解决正在服役的项目的问题,这时应该考虑使用适配器模式。

示例
```java
/**
 * 适配器模式：类适配模式
 */
public class Adapter extends targetClass implements CAdaptee {

    public String readXmlByStream() {
        return readXml();
    }
}


/**
 * 适配器模式：对象适配模式
 */
class Adapter2 implements CAdaptee {
    // 该对象的方法适配我所需求
    targetClass tc = new targetClass();


    public String readXmlByStream() {
        return tc.readXml();
    }
}

interface CAdaptee {
    String readXmlByStream();
}

class targetClass {
    public String readXml() {
        return "we can read readXml";
    };
}

/**
 * 让这个工具也能读取xml格式数据
 */
class ReadDataTool extends Adapter2{
    public String readJson() {
        return "i can read json data!";
    }

    @Override
    public String readXmlByStream() {
        return super.readXmlByStream();
    }
}
```