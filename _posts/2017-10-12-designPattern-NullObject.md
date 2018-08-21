---
layout: post
title: '设计模式之空对象模式'
subtitle: '空对象模式'
date: 2017-10-11
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 行为型模式
#### 空对象模式
在空对象模式（Null Object Pattern）中，一个空对象取代NULL对象实例的检查。Null对象不是检查空值，而是反应一个不做任何动作的关系。这样的 Null 对象也可以在数据不可用的时候提供默认的行为。

**优点** 

**缺点**

**使用场景**：  

- 需要检查空值的地方

示例
在空对象模式中，我们创建一个指定各种要执行的操作的抽象类和扩展该类的实体类，还创建一个未对该类做任何实现的空对象类，该空对象类将无缝地使用在需要检查空值的地方
```java
ublic class NullObjectPattern {

    public static void main(String[] args) {
        NullObjectPattern nullObjectPattern = new NullObjectPattern();

        Factory factory = nullObjectPattern.new Factory();
        System.out.println(factory.getSubject("haha").getName());
        System.out.println(factory.getSubject("PE").getName());
    }

     class Factory {
        public final String[] names = {"PE","MATH","LITERATURE","POLITICAL"};

        public Subject getSubject(String subject) {
            for (String name : names) {
                if (name.equals(subject)) {
                    return new SecondarySubject(subject);
                }
            }

            return new NullSubject();
        }
    }


    class NullSubject extends Subject {
        @Override
        boolean isNull() {
            return true;
        }

        @Override
        String getName() {
            return "not avilable subject";
        }
    }

    class SecondarySubject extends Subject {
        SecondarySubject( String name) {
            this.name = name;
        }

        @Override
        boolean isNull() {
            return false;
        }

        @Override
        String getName() {
            return name;
        }
    }

    abstract class Subject {
        String name;
        abstract boolean isNull();
        abstract String getName();
    }
}
```