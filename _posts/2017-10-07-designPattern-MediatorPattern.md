---
layout: post
title: '设计模式之中介者模式'
subtitle: '中介者模式'
date: 2017-10-03
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 行为型模式
#### 中介者模式
中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护

**优点** 

- 降低了类的复杂度，将一对多转化成了一对一。
- 各个类之间的解耦。
- 符合迪米特原则。

**缺点**

- 中介者会庞大，变得复杂难以维护
- 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。
- 可能不容易观察运行时的特征，有碍于除错。

**使用场景**：  

- 系统中对象之间存在比较复杂的引用关系，导致它们之间的依赖关系结构混乱而且难以复用该对象。
- 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类



示例
```java
public class MediatorPattern {

    public static void main(String[] args) {
        concreteMediator cm = new concreteMediator();
        ColleagueA a = new ColleagueA("tom", cm);
        ColleagueB b = new ColleagueB("jerry", cm);
        cm.setA(a);
        cm.setB(b);
        a.contact("i have something to b");
        b.contact("i busy!! don't intterupt me");
    }
}

abstract class Mediator {
    public abstract void contact(String content, Colleague coll);
}

class concreteMediator extends Mediator {
    ColleagueA a;
    ColleagueB b;

    public ColleagueA getA() {
        return a;
    }

    public void setA(ColleagueA a) {
        this.a = a;
    }

    public ColleagueB getB() {
        return b;
    }

    public void setB(ColleagueB b) {
        this.b = b;
    }

    @Override
    public void contact(String content, Colleague coll) {
        if (coll == a) {
            b.getMes(content);
        } else {
            a.getMes(content);
        }
    }
}


class ColleagueA extends Colleague {
    public ColleagueA(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void getMes(String mes) {
        System.out.println("collA " + name + "get mes " + mes);
    }

    public void contact(String mes) {
        mediator.contact(mes, this);
    }
}

class ColleagueB extends Colleague {
    public ColleagueB(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void getMes(String mes) {
        System.out.println("collB " + name + "get mes " + mes);
    }

    public void contact(String mes) {
        mediator.contact(mes, this);
    }
}

class Colleague {
    protected String name;
    protected Mediator mediator;

    public Colleague(String name, Mediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }
}
```