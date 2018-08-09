---
layout: post
title: '设计模式之观察者模式'
subtitle: '观察者模式'
date: 2017-10-09
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 观察者模式
---

### 行为型模式
#### 观察者模式
定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新

**优点** 

- 观察者和被观察者是抽象耦合的
- 建立一套触发机制。

**缺点**

- 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间
- 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃
- 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化

**使用场景**：  

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用
- 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度
- 一个对象必须通知其他对象，而并不知道这些对象是谁
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制(JAVA 中已经有了对观察者模式的支持类)

示例
```java
public class ObserverPattern {
    
    public static void main(String[] args) {
        Subject subject = new Subject();
        Observer1 observer1 = new Observer1(subject);
        Observer2 observer2 = new Observer2(subject);
        subject.addObserver(observer1);
        subject.addObserver(observer2);
        subject.setState(10);
        subject.setState(100);
    }
}

class Observer1 extends Observer {
    Observer1(Subject subject) {
        this.subject = subject;
    }

    @Override
    void update() {
        System.out.println("观察者一号观察的对象状态改变为：" + subject.getState());
    }
}

class Observer2 extends Observer {
    Observer2(Subject subject) {
        this.subject = subject;
    }

    @Override
    void update() {
        System.out.println("观察者二号观察的对象状态改变为：" + subject.getState());
    }
}

class Subject {
    private List<Observer> observers = new ArrayList<>();

    private Integer state;

    public Integer getState() {
        return state;
    }

    public void setState(Integer state) {
        this.state = state;
        notifyAllObserver();
    }

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    private void notifyAllObserver() {
        for (Observer observer : observers) {
            observer.update();
        }
    }

}

/**
 * 被观察者
 */
abstract class Observer {
    protected Subject subject;
    void update(){};
}
```