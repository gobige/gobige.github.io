---
layout: post
title: '设计模式之模板模式'
subtitle: '模板模式'
date: 2017-10-13
categories: 设计模式
author: yates
cover: ''
tags: 设计模式
---

### 行为型模式
#### 模板模式
在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行

**优点** 
- 封装不变部分，扩展可变部分
- 提取公共代码，便于维护
- 行为由父类控制，子类实现

**缺点**
- 每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大

**使用场景**：  

- 有多个子类共有的方法，且逻辑相同
- 重要的、复杂的方法，可以考虑作为模板方法

示例
定义一个操作中的算法的骨架，而将一些步骤延迟到子类中,为防止恶意操作，一般模板方法都加上 final 关键词
```java
public class TemplatePattern {
    public static void main(String[] args) {
        cooking tofuOfMapo = new TofuOfMapo();
        tofuOfMapo.cookingFood();

        cooking hotChicken = new HotChicken();
        hotChicken.cookingFood();
    }
}

class TofuOfMapo extends cooking{
    @Override
    void prePareCokie() {
        System.out.println("切豆腐");
        System.out.println("宽油");
    }

    @Override
    void cooking() {
        System.out.println("烧酱汁");
        System.out.println("放豆腐");
    }

    @Override
    void cookied() {
        System.out.println("起锅，装盘");
    }
}

class HotChicken extends cooking{
    @Override
    void prePareCokie() {
        System.out.println("杀鸡，放血，拔毛");
        System.out.println("刀工");
    }

    @Override
    void cooking() {
        System.out.println("宽油");
        System.out.println("烧制");
    }

    @Override
    void cookied() {
        System.out.println("起锅，装盘");
    }
}

abstract class cooking {
    abstract void prePareCokie();

    abstract void cooking();

    abstract  void cookied();

    public final void cookingFood() {
        prePareCokie();
        cooking();
        cookied();
    }
}

```