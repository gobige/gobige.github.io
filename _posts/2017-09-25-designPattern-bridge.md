---
layout: post
title: '设计模式之桥接模式'
subtitle: '桥接模式'
date: 2017-09-25
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---


### 结构型模式
#### 桥接模式
桥接（Bridge）是用于把**抽象化与实现化解耦**，使得二者可以独立变化,通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦

**优点** 

- 抽象和实现的分离
- 优秀的扩展能力
- 实现细节对客户透明

**缺点**

- 桥接模式的引入会增加系统的理解与设计难度，由于**聚合关联关系建立在抽象层**，要求开发者针**对抽象进行设计**与编程

**使用场景**：  

- 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，**避免在两个层次之间建立静态的继承联系**，通过桥接模式可以使它们在抽象层建立一个关联关系。
- 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用
- 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。

示例
```java
public class brigdePattern {
    public static void main(String[] args) {
        xiaoMian xm = new xiaoMian(new hotStyle(), new redColor());
        System.out.println(xm.eat());
        laMian lm = new laMian(new lightStyle(), new greenColor());
        System.out.println(lm.eat());
    }

}

interface style {
    String getName();
}

class hotStyle implements style {
    public String getName() {
        return "hot style";
    }
}

class lightStyle implements style {
    public String getName() {
        return "light style";
    }
}

interface color {
    String getColor();
}

class greenColor implements color {
    public String getColor() {
        return "green";
    }
}

class redColor implements color {
    public String getColor() {
        return "red";
    }
}

abstract class nodle {
    private style style;
    private color color;

    nodle(style style, color color) {
        this.color = color;
        this.style = style;
    }

    public String getStyle() {
        return style.getName();
    }

    public void setStyle(style style) {
        this.style = style;
    }

    public String getColor() {
        return color.getColor();
    }

    abstract String getName();

    abstract String eat();
}

class xiaoMian extends nodle {

    xiaoMian(style style, color color) {
        super(style, color);
    }

    public String getName() {
        return "xiaomian";
    }

    public String eat() {
        return "eat color：" + this.getColor() + "style:" + this.getStyle() + getName();
    }
}

class laMian extends nodle {
    laMian(style style, color color) {
        super(style, color);
    }

    public String getName() {
        return "laMian";
    }

    public String eat() {
        return "eat color：" + this.getColor() + "style:" + this.getStyle() + getName();
    }
}
```