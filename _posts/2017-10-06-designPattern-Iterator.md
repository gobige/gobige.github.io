---
layout: post
title: '设计模式之迭代器模式'
subtitle: '迭代器模式'
date: 2017-10-06
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 行为型模式
#### 迭代器模式
迭代器模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示

**优点** 

- 它支持以不同的方式遍历一个聚合对象
- 迭代器简化了聚合类
- 在同一个聚合上可以有多个遍历
- 在迭代器模式中，增加新的聚合类和迭代器类都很方便，无须修改原有代码

**缺点**

- 由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性

**使用场景**：  

- 访问一个聚合对象的内容而无须暴露它的内部表示
- 需要为聚合对象提供多种遍历方式
- 为遍历不同的聚合结构提供一个统一的接口



示例
```java
public class IteratorPattern {
    public static void main(String[] args) {
        MyContainer container = new MyContainer();
        container.add("str1");
        container.add("str7");
        container.add("str5");
        container.add("str2");
        container.add("str3");

        MyIterator iterator = container.getIterator();

        while (iterator.hashNext()) {
            System.out.println(iterator.next());
        }
    }
}

class MyContainer implements Container {
    private String[] strs = new String[10];
    private int size = 0;

    @Override
    public void add(Object object) {
        strs[size++] = (String) object;
    }

    @Override
    public MyIterator getIterator() {
        return new Iterator();
    }

    private class Iterator implements MyIterator {
        int index = 0;
        @Override
        public boolean hashNext() {
            if (index < strs.length) {
                return true;
            }
            return false;
        }

        @Override
        public Object next() {
            if (hashNext()) {
                return strs[index++];
            }

            return null;
        }
    }
}

interface Container {
    MyIterator getIterator();
    void add(Object obj);
}

interface MyIterator {
    boolean hashNext();
    Object next();
}

```