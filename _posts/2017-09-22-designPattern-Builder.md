---
layout: post
title: '设计模式之建造者模式'
subtitle: '建造者模式'
date: 2017-09-22
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 建造者模式
---


### 创建型模式
#### 建造者模式
适用于多个简单的对象一步一步构建成一个复杂的对象，一些基本部件不会变，而其组合经常变化的情况

**优点**： 1、建造者独立，易扩展。 2、便于控制细节风险
**缺点**： 1、产品必须有共同点，范围有限制。 2、如内部变化复杂，会有很多的建造类。
**使用场景**： 1、需要生成的对象具有复杂的内部结构  2、需要生成的对象内部属性本身相互依赖。

示例
```java
/**
 * 建造者模式
 */
public class BuilderPattern {
    public static void main(String[] args) {
        Meal chickenMeal = MealBuilder.buildChickenMeal();
        chickenMeal.showItems();
        Meal vegeMeal = MealBuilder.buildVegeMeal();
        vegeMeal.showItems();
    }
}

class MealBuilder {

    public static Meal buildChickenMeal() {
        Meal meal = new Meal();
        pepsi pepsi = new pepsi();
        chickenBurger chickenBurger = new chickenBurger();
        meal.addItem(pepsi);
        meal.addItem(chickenBurger);

        return meal;
    }

    public static Meal buildVegeMeal() {
        Meal meal = new Meal();
        Coke coke = new Coke();
        VegBurger vegBurger = new VegBurger();
        meal.addItem(coke);
        meal.addItem(vegBurger);

        return meal;
    }
}

class Meal {
    private List<Item> items = new ArrayList<Item>();

    public void addItem(Item item) {
        items.add(item);
    }

    public Double getCost() {
        BigDecimal cost = new BigDecimal("0.00");

        for (Item item : items) {
            cost = cost.add(new BigDecimal(item.price()== null?"0.00":item.price().toString()));
        }

        return cost.doubleValue();
    }

    public void showItems () {
        for (Item item : items) {
            System.out.println(item.toString());
        }
    }
}

class Coke extends ColdDrink {
    @Override
    public String name() {
        return "coke";
    }

    @Override
    public Double price() {
        return 3.50;
    }

    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}


class pepsi extends ColdDrink {
    @Override
    public String name() {
        return "pepsi";
    }

    @Override
    public Double price() {
        return 3.00;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

class chickenBurger extends Burger {
    @Override
    public String name() {
        return "chicken burger";
    }

    @Override
    public Double price() {
        return 11.2;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

class VegBurger extends Burger {
    @Override
    public String name() {
        return "vegBurger";
    }

    @Override
    public Double price() {
        return 5.5;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

/**
 * 冷饮
 */
abstract class ColdDrink implements Item {
    @Override
    public Packing pack() {
        return new Bottle();
    }
}

/**
 * 汉堡
 */
abstract class Burger implements Item{
    @Override
    public Packing pack() {
        return new Wrapper();
    }
}

/**
 * 组件接口
 */
interface Item{
    String name();
    Double price();
    Packing pack();
}

/**
 * 盒装
 */
class Wrapper implements Packing {
    @Override
    public String pack() {
        return "wrapper";
    }
}

/**
 * 瓶装
 */
class Bottle implements Packing {
    @Override
    public String pack() {
        return "bottle";
    }
}
/**
 * 包装
 */
interface Packing {
    String pack();
}
```
