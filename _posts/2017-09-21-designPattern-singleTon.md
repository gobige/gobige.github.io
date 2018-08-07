---
layout: post
title: '设计模式之单例模式'
subtitle: '单例模式'
date: 2017-09-21
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 单例模式
---


### 创建型模式
#### 单例模式
该模式涉及到一个单一的类，该类**负责创建自己的对象**，同时确保只有**单个对象被创建**。这个类提供了一种访问其**唯一的对象的方式**，可以直接访问，不需要实例化该类的对象

**优点**： 1、在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。2、避免对资源的多重占用（比如写文件操作）。

**缺点**：没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

**使用场景**：某个应用，网站的一些不常更新的配置，单独从数据库取出，放入内存中，进行配置的读取

单例模式按加载顺序又可分为饿汉模式，懒汉模式

**饿汉式**
```java
class HungrySingleTon {
    private HungrySingleTon(){}

    private static HungrySingleTon hungrySingleTon = new HungrySingleTon();

    public static HungrySingleTon getInstance() {
        return hungrySingleTon;
    }
}
```

**懒汉式**
```java
class lazySingleTonOfDcl {
    private lazySingleTonOfDcl(){}
    // volatile 关键字，限制cpu对指令重排序
    private static volatile lazySingleTonOfDcl singleTon = null;

    public static lazySingleTonOfDcl getInstance() {
        if (singleTon == null) {
            synchronized (lazySingleTonOfDcl.class) {
                if (singleTon == null) {
                    singleTon = new lazySingleTonOfDcl();
                }
            }
        }
        return singleTon;
    }
}
```
在懒加载单例对象时候，由于Java中**new一个对象**并不是一个**原子操作**，编译时singleton = new Singleton(); 语句会被转成多条汇编指令，它们大致做了3件事情

- 给 Singleton 类的实例分配内存空间； 
- 调用私有的构造函数 Singleton()，初始化成员变量； 
- 将 singleton 对象指向分配的内存（执行完此操作 singleton 就不是 null了）

由于Java编译器允许处理器乱序执行（处理器会根据语句执行效率进行**指令重排序**），以及JDK1.5之前的旧的Java内存模型中Cache、寄存器到主内存回写顺序的规定，上面步骤2)和3)的执行顺序是无法确定的，可能是 1) → 2) → 3) 也可能是 1) → 3) → 2) 。如果是后一种情况，在线程 A 执行完步骤 3) 但还没完成 2) 之前，被切换到线程 B 上，此时线程 B 对singleton 第1次判空结果为false，直接取走了singleton使用，但是构造函数却还没有完成所有的初始化工作，就会出错，也就是DCL失效问题。这时我们在加上volatile关键字就能解决这个问题

另外还有其他实现单例模式的方式，如下图
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-20-designPattern/1.png)