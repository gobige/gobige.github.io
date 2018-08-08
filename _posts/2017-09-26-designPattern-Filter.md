---
layout: post
title: '设计模式之原型模式'
subtitle: '过滤器模式'
date: 2017-09-26
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 过滤器模式
---


### 结构型模式
#### 过滤器模式
过滤器模式使用不同的标准来过滤一组对象，通过逻辑运算以解耦的方式把它们连接起来

示例
```java
/**
 * 标准接口
 */
interface Criteria {
    List<Apple> MaterialCriteria(List<Apple> apples);
}

/**
 * 能做饮料的苹果标准
 */
class AppleDrink implements Criteria {
    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> appleDrinks = new ArrayList<>();
        if (!CollectionUtils.isEmpty(apples)) {
            for (Apple apple : apples) {
                if ("green".equals(apple.getColor()) && 2 > apple.getWight()) {
                    appleDrinks.add(apple);
                }
            }
        }

        return appleDrinks;
    }
}

/**
 * 能做披萨的苹果标准
 */
class ApplePizza implements Criteria {
    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> applePizzas = new ArrayList<>();
        if (!CollectionUtils.isEmpty(apples)) {
            for (Apple apple : apples) {
                if ("Brazil".equals(apple.getCountry())) {
                    applePizzas.add(apple);
                }
            }
        }

        return applePizzas;
    }
}

/**
 * 符合两个标准的苹果
 */
class AndCriteria implements Criteria {
    private Criteria criteria;
    private Criteria oherCriteria;

    AndCriteria(Criteria criteria,Criteria oherCriteria) {
        this.criteria = criteria;
        this.oherCriteria = oherCriteria;
    }

    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> criteriaApples = criteria.MaterialCriteria(apples);
        List<Apple> andApples = oherCriteria.MaterialCriteria(criteriaApples);

        return andApples;
    }
}

/**
 * 两个标准都不符合的苹果
 */
class OrCriteria implements Criteria {
    private Criteria criteria;
    private Criteria oherCriteria;

    OrCriteria(Criteria criteria,Criteria oherCriteria) {
        this.criteria = criteria;
        this.oherCriteria = oherCriteria;
    }

    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> criteriaApples = criteria.MaterialCriteria(apples);
        List<Apple> otherCriterApples = oherCriteria.MaterialCriteria(apples);
        List<Apple> orApples = new ArrayList<>();

        for (Apple apple : criteriaApples) {
            if (!otherCriterApples.contains(apple)) {
                orApples.add(apple);
            }
        }
        return orApples;
    }
}
```