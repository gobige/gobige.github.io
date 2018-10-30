---
layout: post
title: '设计模式之解释器模式'
subtitle: '解释器模式'
date: 2017-10-05
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 行为型模式
#### 解释器模式
解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等

**优点** 

- 可扩展性比较好，灵活。
- 增加了新的解释表达式的方式
- 易于实现简单语法

**缺点**

- 可利用场景比较少。JAVA 中如果碰到可以用 expression4J 代替
- 对于复杂的语法比较难维护。
- 解释器模式会引起类膨胀。
- 解释器模式采用递归调用方法。

**使用场景**：  

- 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树。
- 一些重复出现的问题可以用一种简单的语言来进行表达。
- 一个简单语法需要解释的场景



示例
```java
public class InterpreterPattern {
    public static void main(String[] args) {
        System.out.println("hello,this is bbc news is news?:" + InterpreterPattern.setNewsWord().interpret("hello,this is bbc news"));
        System.out.println("1 hour 30 mills is time?:" + InterpreterPattern.setTimeWord().interpret("1 hour 30 mills"));

    }

    public static Explain setNewsWord() {
        Explain contextExplain = new ContextExplain("news");
        Explain contextExplain2 = new ContextExplain("report");

        return new OrExplain(contextExplain,contextExplain2);
    }

    public static Explain setTimeWord() {
        Explain contextExplain = new ContextExplain("minute");
        Explain contextExplain2 = new ContextExplain("hour");

        return new AndExplain(contextExplain,contextExplain2);
    }
}

class AndExplain implements Explain {
    private Explain orExplain1;
    private Explain orExplain2;

    AndExplain(Explain orExplain1, Explain orExplain2) {
        this.orExplain1 = orExplain1;
        this.orExplain2 = orExplain2;
    }


    @Override
    public boolean interpret(String context) {
        return orExplain1.interpret(context) && orExplain2.interpret(context);
    }
}

class OrExplain implements Explain {
    private Explain orExplain1;
    private Explain orExplain2;

    OrExplain(Explain orExplain1, Explain orExplain2) {
        this.orExplain1 = orExplain1;
        this.orExplain2 = orExplain2;
    }


    @Override
    public boolean interpret(String context) {
        return orExplain1.interpret(context) || orExplain2.interpret(context);
    }
}

class ContextExplain implements Explain {
    private  String data;

    ContextExplain(String data) {
        this.data = data;
    }

    @Override
    public boolean interpret(String context) {
        if (context.contains(data)) {
            return true;
        }

        return false;
    }
}

interface Explain {
    boolean interpret(String context);
}

```