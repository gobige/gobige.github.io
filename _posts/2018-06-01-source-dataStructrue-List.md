---
layout: post
title: 'java看源码系列-队列'
subtitle: 'openjdk queue的实现'
date: 2017-07-29
categories: 个人博客
author: yates
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: githubpages
---
**Collection接口**
定义了所有子类的基础方法
- size() 集合元素数量
- isEmpty() 集合是否为空
- contains(Object o) 集合是否包含某元素
- iterator() 返回该集合的迭代器
- toArray() 集合中元素返回对象数组形式
- toArray(T[] a) 转换为返回a对象类型的数组形式
- add(E e) 增加一个元素
- remove(Object o)删除此列表中指定位置的元素， 返回从中删除的元素
- containsAll(Collection<?> c) 是否包含某个集合里所有元素
- addAll(Collection<? extends E> c) 添加某集合到该集合
- removeAll(Collection<?> c) 在该集合中删除某集合
- removeIf(Predicate<? super E> filter) 通过过滤条件判断是否存在该元素，并删除(1.8新增默认方法)
- retainAll(Collection<?> c) 保留在该集合中存在的元素
- clear() 清除集合所有元素
- equals(Object o) 是否等于集合
- hashCode() 获取该集合hashcode
- spliterator() (1.8新增默认方法)
- stream() 返回该集合串行流，（1.8新增默认方法）
- parallelStream() 返回该集合并行流，（1.8新增默认方法）

**AbstractCollection抽象类**
部分实现了collection接口的一些方法

**List接口**
在collection接口的基础上扩展了一下接口
- addAll(int index, Collection<? extends E> c)  (1.8新增默认方法)
- sort(Comparator<? super E> c)  (1.8新增默认方法)
- get(int index) 通过数组下标获取元素
- set(int index, E element) 通过数组下标替换某元素
- add(int index, E element) 将指定元素插入此列表中的指定位置，将当前位置的元素任何后续元素向右移动
- remove(int index) 通过数组下标移除元素
- indexOf(Object o) 返回某元素第一次出现数组下标
- lastIndexOf(Object o) 返回某元素最后一次出现数组下标
- listIterator() 
- listIterator(int index);
- subList(int fromIndex, int toIndex) 返回该列表指定范围的列表视图 

**AbstractList抽象类**
在AbstractCollection抽象类基础上继续实现list接口里面的方法

**ArrayList**
ArrayList的继承关系图

![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-06-01-source-dataStructrue-List/1.png)

arrayList数据结构是对象数组
```java
Object[] elementData; 
```

它的三种构造方法，都会开辟一片内存来放数组
```java
ArrayList(int initialCapacity) 
ArrayList()
ArrayList(Collection<? extends E> c) 
```

使用来使整个列表大小和元素个数一样 modacout+1
```java
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

改变此列表的容量
```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        ? 0
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

clone 该列表（浅度克隆）
```java
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
```

得到该列表的数组形式（开辟一个新的内存，存取该数据，还是浅度克隆）
```java
  public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```  

通过数组下标获取元素
```java 
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

通过数组下标获取元素 **某些情况下查询速度很快**
```java 
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

增加一个元素,先对列表扩容（调用array.copy浅度克隆，modacout+1） **元素可重复，可为null 插入相对耗时**
```
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

删除指定位置元素 增加modcount 把index位置以右元素左移，把最后元素赋值为null **删除相对耗时**
```
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

删除某个元素 modcount+ 遍历删除，内存地址一样元素 把该元素以右元素左移
```
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```

清除列表元素  modcount++ 遍历赋值元素为null，
```
   public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```
**Vector**
Vector的继承关系图
![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-06-01-source-dataStructrue-List/2.png)

**LinkedList**
LinkedList的继承关系图
![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-06-01-source-dataStructrue-List/3.png)