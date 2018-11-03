---
layout: post
title: 'java8之steam流'
subtitle: 'java8之steam流的一些用法'
date: 2018-06-18
categories: javaapi
author: yates
cover: ''
tags: java
---

### 前言
java8新增的功能中，增加了stream流API，那么stream流究竟有什么作用了，该怎么使用呢？

### 什么是流
所谓流就是数据的渠道，流代表的是一个对象的序列，它和java IO流是两个不同的东西，这里的流指的是某个流类型的对象。

- 终端操作：终端操作会消费流，一个被消费后的流是**不能再次利用**的，如：min max

- 中间操作：中间操作会对流进行处理，会**产生另一个流**，但是中间操作**不是立即发生**的，只有当中间流执行完终端操作后，中间操作才会发生，这种延迟的行为使得流api**高效**的执行 
	- 中间操作分为**无状态操作**和**有状态操作**，无状态操作指的是处理流中元素的时候，对当前元素进行单独处理，比如**谓词过滤**，而有状态操作中，某个元素的处理会依赖其他元素

stream常用的方法
```java
// 产生一个新流，包含调用流中满足predicate的谓词元素 中间操作
Stream<T> filter(Predicate<? super T> predicate);

// 产生一个新流，对调用流中元素映射mapper 中间操作
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

// 产生一个新的intStream流，对调用流中元素映射mapper 中间操作
IntStream mapToInt(ToIntFunction<? super T> mapper);

// 产生一个按一定方式排序的新流 中间操作
 Stream<T> sorted(Comparator<? super T> comparator);
 
// 遍历流中元素 终端操作
void forEach(Consumer<? super T> action);

// 获取流中最大，最小值 终端操作
Optional<T> max(Comparator<? super T> comparator);
```

### 缩减操作
用流API的术语来形容，缩减操作就是把一个流缩减为一个值，先来看看stream接口中的reduce方法
```java
Optional<T> reduce(BinaryOperator<T> accumulator);
T reduce(T identity, BinaryOperator<T> accumulator);
<U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner);
```
第一个方法返回要给optionanl对象，第二个方法返回的是T类型，就是流中元素类型；accumulator的类型是扩展了BiFunction函数式接口
```java
    R apply(T t, U u);
```
apply对t参数和u参数应用到同一个函数上，并返回结果r，而BinaryOperator扩展BiFunction时所有参数都是T

所以通过reduce缩减操作后得到的结果取决于具体使用reduce方法
### 缩减操作三个约束
- 无状态 流中每个元素能够被单独处理
- 不干预 操作数不会改变数据源
- 关联性 给定一个关联运算符，一些列操作中，处理次序的先后是无关的

```java
List<Integer> s = new ArrayList<>();
s.add(1);s.add(4);
s.add(5);s.add(6);
s.add(3);s.add(2);
Optional sum = s.stream().reduce((a,b) -> a + b);

if (sum.isPresent()) {
    System.out.println("list sum is " + sum.get());
}

Integer sum2 = s.stream().reduce(0, (a,b) -> a+b);
System.out.println("list sum is " + sum2);

Optional sum3 = s.stream().reduce((a, b) -> a * b);
System.out.println("list sum is " + sum3.get());

 Integer sum4 = s.stream().reduce(1, (a,b) -> a*b);
System.out.println("list sum is " + sum4);
```
结果：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-06-18-java_stream/1.png)


### 并行流
获取方式：1通过**parallel()**方法获取流 2通过collection接口提供的**parallelStream**方法获取并行流
上面的例子使用并行流操作：
```java
 List<Integer> s = new ArrayList<>();
    s.add(1);
    s.add(2);
    s.add(3);
    s.add(4);
    s.add(5);
    s.add(6);
    Optional sum = s.parallelStream().reduce((a,b) -> a + b);

    if (sum.isPresent()) {
        System.out.println("list sum is " + sum.get());
    }

    Integer sum2 = s.parallelStream().reduce(0, (a,b) -> a+b);
    System.out.println("list sum is " + sum2);

    Optional sum3 = s.parallelStream().reduce((a, b) -> a * b);
    System.out.println("list sum is " + sum3.get());

     Integer sum4 = s.parallelStream().reduce(1, (a,b) -> a*b);
    System.out.println("list sum is " + sum4);
```
结果是一样的，应用到并行流的操作都必须符合缩减操作的三个约束条件，使得并行流上获得结果和顺序流是一样的

在使用并行流的时候，如果集合和数组中元素是有序的出来的流也是有序的，如果时无序的，出来的流也是无序的，有时候流时无序的能获得性能上的提升，unordered()方法将流转换为无序流

- foreach方法不会保留并行流的顺序，foreach采用的时获取迭代器的方式进行遍历，跟数据结构无关
- 如果数据结构时数组结构，可以采用for;如果数据结构时链表结构，那么果断采用foreach，提高遍历效率


### 映射
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```
R表示新的流元素类型，T表示映射前流元素类型，map获取流中元素某个小元素类型，并返回这个类型的一个新流

获取所有苹果的重量总和
```java
List<Integer> wight = apples.stream().map(Apple::getWight).collect(Collectors.toList());
```

### 收集
Collectors类是一个最终类，里面提供大量静态收集器方法，可借此实现各种复杂功能

收集苹果的名和产地
```java
List<String> names = apples.stream().map(Apple::getName).collect(Collectors.toList());
Set<String> nameSets = apples.stream().map(Apple::getCountry).collect(Collectors.toSet());
```

TODO 未完待续

### 流使用例子
```java
// List中去除某值形成List
List<Apple> apples = new ArrayList<Apple>();
List<Apple> apples2 = new ArrayList<Apple>();
Apple a = new Apple();
apples.add(a);
List<Integer> wight = apples.stream().map(Apple::getWight).collect(Collectors.toList());
// List 转map
Map<Integer, String> map = apples.stream().collect(Collectors.toMap(Apple::getWight, Apple::getCountry));

// List中字段求和
Integer test = apples.stream().collect(Collectors.summingInt(Apple::getWight));
// 获取比当前时间大的最小的时间
List<Date> dateList = new ArrayList<>();

Date mindate = dateList.stream().filter(date -> date.getTime() > System.currentTimeMillis()).min(Comparator.comparing(Date::getTime)).get();

// 求交集
List<Apple> list3 = apples.stream().filter(item -> apples2.contains(item)).collect(Collectors.toList());
// 球最小
Apple minAppple = apples.stream().min(Comparator.comparing(Apple::getWight)).get();
// 求差集
List<Apple> list4 = apples.stream().filter(item -> !apples2.contains(item)).collect(Collectors.toList());
// 求去重并集
apples.addAll(apples2);
apples.stream().distinct().collect(Collectors.toList());

//循环输出
apples.stream().forEach(System.out::println);

// 分组
Map<String, List<Apple>> result2 = apples.stream().collect(Collectors.groupingBy(Apple::getCountry));
```
```