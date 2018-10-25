---
layout: post
title: '设计模式之策略模式'
subtitle: '策略模式'
date: 2017-10-11
categories: 设计模式
author: yates
cover: ''
tags: 设计模式
---

### 行为型模式
#### 策略模式
在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改

**优点** 
- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好。

**缺点**
- 策略类会增多
- 所有策略类都需要对外暴露

**使用场景**：  

- 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为
- 一个系统需要动态地在几种算法中选择一种
- 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现

示例
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法
```java
public class StrategyPattern {

    public static void main(String[] args) {
        Army army1 = new Army(new OffensiveStrategy());
        Army army2 = new Army(new RetreatStrategy());

        army1.excuteStrategy();
        army2.excuteStrategy();
    }
}

class Army {
    private Strategy strategy;

    Army(Strategy strategy) {
        this.strategy = strategy;
    }

    void excuteStrategy() {
        strategy.doOperation();
    }
}

/**
 * 进攻策略
 */
class OffensiveStrategy implements Strategy {
    @Override
    public void doOperation() {
        System.out.println("执行进攻策略");
    }
}

/**
 * 撤退策略
 */
class RetreatStrategy implements Strategy {
    @Override
    public void doOperation() {
        System.out.println("执行撤退策略");
    }
}

interface Strategy {
    void doOperation();
}
```