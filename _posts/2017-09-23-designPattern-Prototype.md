---
layout: post
title: '设计模式之原型模式'
subtitle: '原型模式'
date: 2017-09-23
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---


### 创建型模式
#### 原型模式
原型模式用于创建**重复的对象**，同时又能保证**性能**

**优点**

- 性能提高。 
- 逃避构造函数的约束。

**缺点**

- 配备**克隆**方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。
- 必须实现 Cloneable 接口。

**使用场景**

- 资源优化场景。 
- 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。 
- 性能和安全要求的场景。 
- 通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。 
- 一个对象多个修改者的场景。 
- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。 
- 在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。

示例
```java
/**
 * 原型模式
 */
public class PrototypePattern {
    public static void main(String[] args) {
        BallCache.loadCache();
        try {
            BallCache.getBall("footBall");
        }catch (Exception e){
            e.getMessage();
        }

    }
}

class BallCache {
    private static Hashtable<String,Ball> hashtable = new Hashtable();


    public static Ball getBall(String ballName) throws Exception{
        return (Ball)hashtable.get(ballName).clone();
    }

    public static void loadCache() {
        hashtable.put("footBall",new FootBall());
        hashtable.put("basketBall",new BasketBall());
    }

}

class FootBall extends Ball {
    FootBall() {
        this.wight = 1.1;
        this.color = "white";
    }
}
class BasketBall extends Ball {
    BasketBall() {
        this.wight = 2.1;
        this.color = "red";
    }
}

abstract class Ball implements Cloneable {
    protected String name;
    protected Double wight;
    protected String color;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getWight() {
        return wight;
    }

    public void setWight(Double wight) {
        this.wight = wight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```