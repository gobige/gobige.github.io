---
layout: post
title: '设计模式之原型模式'
subtitle: '装饰器模式'
date: 2017-09-28
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 装饰器模式
---


### 结构型模式
#### 装饰器模式
装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构，创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能

**优点** 

- 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能

**缺点**

- 多层装饰比较复杂

**使用场景**：  

- 扩展一个类的功能
- 动态增加功能，动态撤销

示例
```java
/**
 * 装饰者模式
 */
public class DecoratorPattern {
    public static void main(String[] args) {
        Person england = new EnglandMan();
        england.speak();
        Person china = new Chinese();
        china.speak();
        Person superman = new FlyPersonDecorator(new Chinese());
        superman.speak();
    }
}

interface Person {
    void speak();
}

class FlyPersonDecorator extends PersonDecorator {
    FlyPersonDecorator(Person decoratePerson) {
        super(decoratePerson);
    }

    @Override
    public void speak() {
        super.speak();
        setWing();
    }

    private void setWing() {
        System.out.println("i can fly");
    }
}

abstract class PersonDecorator implements Person {
    protected Person decoratePerson;

    PersonDecorator(Person decoratePerson) {
        this.decoratePerson = decoratePerson;
    }

    @Override
    public void speak() {
        decoratePerson.speak();
    }
}

class EnglandMan implements Person {

    @Override
    public void speak() {
        System.out.println("i will speak english");
    }
}

class Chinese implements Person{
    @Override
    public void speak() {
        System.out.println("i will speak chinese");
    }
}
```