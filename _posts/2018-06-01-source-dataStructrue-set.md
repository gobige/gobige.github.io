---
layout: post
title: 'java看源码系列-set集合'
subtitle: 'openjdk HashSet LinkedHashSet TreeSet的实现'
date: 2018-06-01
categories: 个人博客
author: yates
cover: ''
tags: 源码
---

**set接口定义了所需实现方法**

- int size(); 获取集合大小
- boolean isEmpty(); 集合是否为空
- boolean contains(Object o); 是否包含某元素
- Iterator<E> iterator(); 返回迭代器
- Object[] toArray(); 展示为数组对象形式
- <T> T[] toArray(T[] a);
- boolean add(E e); 向集合中增加某个元素
- boolean remove(Object o); 删除某个元素
- boolean containsAll(Collection<?> c); 是否包含所有给定集合的元素
- boolean addAll(Collection<? extends E> c); 向集合中添加所给予集合中所有元素
- boolean retainAll(Collection<?> c); 保留给定集合中所拥有的元素
- boolean removeAll(Collection<?> c); 移除所给予集合中的元素
- void clear(); 清空集合
- boolean equals(Object o); 
- int hashCode();

**AbstractSet抽象类**
部分的实现,重写了set接口的方法 hashCode equals removeAll

**HashSet类**
继承AbstractSet,实现了set接口的方法
hashset的

