---
layout: post
title: '大数据处理相关算法'
subtitle: 'bitSet，布隆过滤器等'
date: 2017-09-12
categories: 算法
author: yates
cover: 'http://cctv.com'
tags: 大数据处理相关
---

## bitSet
java bitSet类是一种特殊类型的**数组**，用于存储**位值**，提供的方法本质上是位运算，所以对于一些数据的处理特别快。
- bitSet 数组中只会有0和1
- bitSet不是线程安全的


### 应用场景
- 统计一组数据中没有出现过的数；将这组数据映射到BitSet，然后遍历BitSet，对应位为0的数表示没有出现过的数据。

- 对数据进行排序。将数据映射到BitSet，遍历BitSet得到的就是有序数据。

- 在内存对数据进行压缩存储等等。一个GB的内存空间可以存储85亿多个数，可以有效实现数据的压缩存储，节省内存空间开销。

### 示例

```java
// 数据准备
int i = 0;
List<Apple> apples = new ArrayList<>();
while (i < 10000000) {
    Apple apple = new Apple();
    if (i % 4 == 0) {
        apple.setCountry("china");
    }
    if (i % 6 == 0) {
        apple.setWight(10);
    }
    apple.setSeqNo(i);
    apples.add(apple);
    i++;
}

BitSet chinaBitSet = new BitSet();
for (Apple apple : apples) {
    if ("china".equals(apple.getCountry())) {
        chinaBitSet.set(apple.getSeqNo());
    }
}

BitSet wightSet = new BitSet();
for (Apple apple : apples) {
    if (Integer.valueOf(10).equals(apple.getWight())) {
        wightSet.set(apple.getSeqNo());
    }
}

// bitset求交集
long startTimeOfBitSet = System.currentTimeMillis();
wightSet.and(chinaBitSet);
long endTimeOfBitSet = System.currentTimeMillis();
System.out.println("bitset method cost time---" + (endTimeOfBitSet - startTimeOfBitSet));

// 正常求交集
long startTimeOfNormal = System.currentTimeMillis();
apples = apples.stream().filter(a -> "china".equals(a.getCountry()) && Integer.valueOf(10).equals(a.getWight())).collect(Collectors.toList());
long endTimeOfNormal = System.currentTimeMillis();
System.out.println("normal method cost time---" + (endTimeOfNormal - startTimeOfNormal));

```

### bitSet局限性
数字分布不均匀很浪费空间
只能表示正整数
不容纳冲突

## 布隆过滤器
对于不同字符串问题，通过hash后得到的hash值可能一样，那么就无法判断这个字符串是否存在。布隆过滤器使用多个hash函数对该
字符串进行hash得到不同hash值存入数组中，表示为1。如果下次判断这个字符串是否存入时，同时判断多个hash后得到的值是否都为1，
如果有**其中一个为0**，则这个字符串**一定不存在**；如果**全为1**，这个字符串**不一定存在**。最大限度的避免了hash冲突


### 应用场景

1. 判断某个key是否存在,如：黑名单，邮件过滤，URL访问

### 布隆过滤器局限性
有一定的误判性，取决于hash函数的质量和数量。跟过滤性能在一定程度上成反比。

### 未完待续....