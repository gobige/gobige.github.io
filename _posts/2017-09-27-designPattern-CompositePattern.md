---
layout: post
title: '设计模式之组合模式'
subtitle: '组合模式'
date: 2017-09-27
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---


### 结构型模式
#### 组合模式
组合模式（Composite Pattern），又叫**部分整体**模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次，创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式

**优点** 

- 高层模块调用简单
- 节点自由增加

**缺点**

- 在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则

**使用场景**：  

- 部分、整体场景，如树形菜单，文件、文件夹的管理

示例
```java
class subject {
    private String content;
    private String author;
    private String time;

    private Collection<comment> comments;
}

class comment {
    private String content;
    private String replyName;
    private String time;

    private Collection<reply> replies;
}

class reply {
    private String content;
    private String replyName;
    private String time;
}
```