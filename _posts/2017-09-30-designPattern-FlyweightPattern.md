---
layout: post
title: '设计模式之原型模式'
subtitle: '享元模式'
date: 2017-09-30
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 享元模式
---

### 结构型模式
#### 享元模式
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能，享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象

**优点** 

- 大大减少对象的创建，降低系统的内存，使效率提高

**缺点**

- 提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱

**使用场景**：  

-  1、系统有大量相似对象。 2、需要缓冲池的场景。


示例
```java
public class Facade {
    registerClass rc = new registerClass();
    MathClass mc = new MathClass();
    notifyClass nc = new notifyClass();

    public void registerWeibo() {
        rc.register();
        mc.matchFriend();
        nc.notifyUser();
    }
}

class registerClass {
    public String register() {
        return "注册账号";
    }
}

class MathClass {
    public String matchFriend() {
        return "匹配朋友";
    }
}

class notifyClass {
    public String notifyUser() {
        return "短信通知";
    }
}
```