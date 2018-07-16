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

删除指定位置元素 增加modcount 把index位置以右元素左移，把最后元素赋值为null **删除相对耗时 这也是为什么删除，指定位置增加操作要使用迭代器来进行才能保证删除，增加的的正确性**
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

删除某个元素 modcount+ 遍历删除，内存地址一样元素 把该元素以右元素左移 最后一个元素赋值为null
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

复制一个列表到另一个列表 浅度克隆
```java 
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```

复制一个列表到另一个列表的指定位置， 指定位置元素向右便移动添加列表size位置
```java
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```

移除或保留给定列表中存在于该列表的元素 （这里取巧 在原有列表上操作，不开辟新内存，节约了空间浪费）
```java
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
     public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
```

序列化该列表，序列化期间不能对其进行操作，否者会报同步异常
```java
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```


反序列化该列表，
```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

获取该列表的迭代器 每次调用该方法都会生成一个新的迭代对象
```
iterator()

 private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```
itr内部类有三个全局变量cursor，lastRet，expectedModCount，他的next和remove方法调用都会检查 mod的次数，也就是说 在列表在创建迭代器的后，进行列表迭代时，如果有其他线程也在对该列表做操作，那么
那么能够快速的返回一个同步的异常

ListItr内部类是itr的升级版，提供反向迭代，并且提供，set和add方法

返回该列表指定区域的视图
```java
    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
```
SubList内部类映射arraylist指定区域的元素，并提供set，get，size，add，remove，removerange等元素操作方法，由于并没有开辟一块新的内存，所以sublist的所有操作结果都会呈现在arraylist上面，同样sublist也有iterator，甚至sublist，而且也是同步快速返回失败的

使用指定的比较器对列表进行排序 同样也是快速失败的
```java
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

**Vector**
Vector的继承关系图
![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-06-01-source-dataStructrue-List/2.png)

**LinkedList**
LinkedList的继承关系图
![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-06-01-source-dataStructrue-List/3.png)