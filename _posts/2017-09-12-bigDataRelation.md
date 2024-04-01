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

# bitSet
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

# 布隆过滤器

布隆过滤器（Bloom Filter）是1970年由布隆提出的，它实质上是一个很长的二进制向量和一系列随机映射函数 (Hash函数)。


### 应用场景

作用：它是一个空间效率高的概率型数据结构，用来告诉你:一个元素**一定不存**在或者**可能存在**。

1. 判断某个key是否存在,如：网络爬虫,黑名单，邮件过滤，URL访问
2. 预防缓存穿透：布隆过滤器快速判断数据是否存在，避免通过查询数据库来判断数据是否存在。


### 优点
相比于其它的数据结构，布隆过滤器在空间和时间方面都有巨大的优势。布隆过滤器存储空间和插入/查询时间都是常数（即hash函数的个数）。
Hash 函数相互之间没有关系，方便由硬件并行实现。
布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势。
布隆过滤器可以表示全集，其它任何数据结构都不能。

   
### 结构
布隆过滤器实现原理就是一个超大位数的数组和多个不同Hash算法函数。假设位数组的长度为 m，哈希函数的个数为 k。如下图，一个长度16位的数组，3个不同Hash算法函数，数组里面存储的是 bit 位，只放 0 和 1，初始为 0



### 布隆过滤器局限性

#### 误判率

参数
m：布隆过滤器的bit长度。
n：插入过滤器的元素个数。
k：哈希函数的个数。

由误判率公式可知，在**k一定**的情况下，当**n增加时，误判率增加**，**m增加时，误判率减少**

#### 不支持删除


### 布隆过滤器实现方式

#### Guava实现

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>23.0</version>
</dependency>
```

|方法名|功能|参数|返回值|
:---|:---:|:---:|---:|
|put|	添加元素|	put(T object)|	boolean|
|mightContain|	检查元素是否存在|	mightContain(T object)|	boolean|
|copy|	根据此实例创建一个新的BloomFilte|	copy()|	BloomFilter|
|approximateElementCount|	已添加到Bloom过滤器的元素的数量|	approximateElementCount()|	long|
|expectedFpp|	返回元素存在的错误概率|	expectedFpp()|	double|
|isCompatible|	确定给定的Bloom筛选器是否与此Bloom筛选器兼容|	isCompatible(BloomFilterthat)|	boolean|
|putAll|	通过执行的逐位OR将此Bloom过滤器与另一个Bloom过滤器组合|	putAll(BloomFilterthat)|	void|


#### Redis实现

- 开源Redisson(RBloomFilter)
- Redis 4.0 官方提供布隆过滤器插件
- 通过Redis提供的bitMap自己实现

### 未完待续....
