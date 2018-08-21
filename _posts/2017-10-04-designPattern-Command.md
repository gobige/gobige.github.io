---
layout: post
title: '设计模式之命令模式'
subtitle: '命令模式'
date: 2017-10-04
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 行为型模式
#### 命令模式
责任链模式命令模式（Command Pattern）是一种数据驱动的设计模式,请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令

**优点** 

- 降低了系统耦合度。
- 新的命令可以很容易添加到系统中去。

**缺点**

- 使用命令模式可能会导致某些系统有过多的具体命令类

**使用场景**：  

- 认为是命令的地方都可以使用命令模式，比如： 1、GUI 中每一个按钮都是一条命令。 2、模拟 CMD



示例
```java
public class CommandPattern {
    public static void main(String[] args) {
        Stock stock = new Stock("footBall",100);
        AddStock addStock = new AddStock(stock,23);
        ReduceStock reduceStock = new ReduceStock(stock,121);
        Cmd.scanf(addStock);
        Cmd.scanf(reduceStock);
        Cmd.excute();
    }

}

class Cmd {
    private static List<Commond> commonds = new ArrayList<>();

    public static void scanf(Commond commond) {
        commonds.add(commond);
    }

    public static void excute() {
        commonds.stream().forEach(commond -> commond.excute());
    }
}

class Stock {
    private String GoodName;
    private Integer num;

    Stock(String goodName, Integer num) {
        this.GoodName = goodName;
        this.num = num;
    }

    public void add(Integer num) {
        System.out.println(GoodName + " add stock[" + num + "]");
    }

    public void reduce(Integer num) {
        System.out.println(GoodName + " reduce stock[" + num + "]");
    }
}

class AddStock implements Commond {

    private Stock stock;
    private Integer num;

    AddStock(Stock stock,Integer num) {
        this.stock = stock;
        this.num = num;
    }

    @Override
    public void excute() {
        stock.add(num);
    }
}

class ReduceStock implements Commond {

    private Stock stock;
    private Integer num;

    ReduceStock(Stock stock,Integer num) {
        this.stock = stock;
        this.num = num;
    }

    @Override
    public void excute() {
        stock.reduce(num);
    }
}

interface Commond {
    void excute();

```