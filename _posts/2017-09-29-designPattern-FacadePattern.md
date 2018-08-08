---
layout: post
title: '设计模式之外观模式'
subtitle: '外观模式'
date: 2017-09-28
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 外观模式
---

### 结构型模式
#### 外观模式
外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口，这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。

**优点** 

- 1、减少系统相互依赖。 2、提高灵活性。 3、提高了安全性

**缺点**

- 不符合开闭原则，如果要改东西很麻烦，继承重写都不合适。

**使用场景**：  

- 1、为复杂的模块或子系统提供外界访问的模块。 2、子系统相对独立。 3、预防低水平人员带来的风险。


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