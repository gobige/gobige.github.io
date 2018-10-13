---
layout: post
title: 'java看源码系列-列表'
subtitle: 'openjdk arrayList vector linkList的实现'
date: 2018-06-01
categories: 个人博客
author: yates
cover: ''
tags: source
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

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-06-01-source-dataStructrue-List/1.png)

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
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-06-01-source-dataStructrue-List/2.png)

vector有三个变量
```java
protected Object[] elementData; // 存放元素的数组
protected int elementCount;  // 元素个数
protected int capacityIncrement; // 空间每次增长大小
```

构造方法
```java
    public Vector() {
        this(10);
    }
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
```

转换为数组形式（将列表中的数组浅度拷贝到另一个对象数组） **可以看到该方法是加了锁的，不仅这个方法，下面很多对数组进行操作的方法都是加了锁的，所以相比arraylist,vector是线程安全的**
```java
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }
```
有趣的是vector允许设置元素的个数，如果size大于当前列表的size数，则开辟新存储空间扩展size，并赋值扩展的元素为null；若size小于当前列表size数，则保留设置size的列表数，其他的元素删除
```java  
public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }
```

返回列表大小和元素个数
```java
    public synchronized int capacity() {
        return elementData.length;
    }
    public synchronized int size() {
        return elementCount;
    }
    
```

是否包含某个元素
```java
    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }

```

设置某个位置的元素值
```java
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }
```

删除，增加，修改数组中某个位置的元素，视图，迭代器和araylist一样的大同小异的算法，只不过**每个方法操作都是加了锁**
```java
 public synchronized void removeElementAt(int index) 
 public synchronized void insertElementAt(E obj, int index) 
 addElement(E obj)
```


**LinkedList**
LinkedList的继承关系图
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-06-01-source-dataStructrue-List/3.png)

首先我们来看linkList的变量

使用了不同与arraylist和vector数组结构的链表结构，这样的结构使得空间使用上没有一点冗余
```java
transient int size = 0; // 列表大小
transient Node<E> first; // 头部置节点
transient Node<E> last; // 尾部节点
```
Node元素结构
```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

列表头部新增节点 **可以看见linkList也是运行节点元素值为null的**
```java
    public void addFirst(E e)  // 公共方法
    private void linkFirst(E e) { // 内部私有方法 
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

列表尾部新增节点
```java
    public boolean add(E e)   // 公共方法
    public void addLast(E e)  // 公共方法
    void linkLast(E e) {  // 内部私有方法 
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

某个节点前增加节点
```java
linkBefore(E e, Node<E> succ) 
```

去除首节点元素
```java
private E unlinkFirst(Node<E> f)  // 内部私有方法 
removeFirst() // 公共方法
```

去除尾节点元素
```java
unlinkLast(Node<E> l)  // 内部私有方法 
public E removeLast() // 公共方法
```

去除某个节点
```java
E unlink(Node<E> x) 

```

得到首节点或尾节点元素值，如果该节点不存在，则抛异常
```java
public E getLast() 
public E getFirst() 
```

linkList的删除，查找方法都是遍历整个列表，进行对象地址比较，**这种方式相比arrlist和vector来说，某些情况下就很慢了，而删除，增加操作由于是链表结构，不用像arralist那样进行容量扩展和数组赋值，相对来说就比较快**
```java
    public boolean remove(Object o) { // 只会删除第一次碰到的该对象
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

clear方法就和arraylist和vector一样的效率了
```
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

获取某个位置的节点元素，由于是链表结构，所以查询某个位置下面节点元素值，需要遍历，时间复杂度为o(n/2)
```
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
    }
```

查找某个对象元素第一次出现的所在节点位置
```
 public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```


检索节点的元素值，
```
    public E element() {// 获取首节点，不存在会抛出异常
        return getFirst();
    }
    public E peek() { // 获取首节点，不存在不会报错（建议使用该方法）
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
    public E peekFirst() { // 获取首节点，不存在不会报错（建议使用该方法）
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }
    public E peekLast() {// 获取尾节点，不存在不会报错（建议使用该方法）
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```

检索并删除链表节点
```
    public E poll() { // 不存在不会报错（建议使用该方法）
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    public E pollFirst() {// 不存在不会报错（建议使用该方法）
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    public E pollLast() {// 不存在不会报错（建议使用该方法）
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

    public E remove() {// 不存在会抛出异常
        return removeFirst();
    }
    public E pop() {
        return removeFirst();
    }

```

增加节点
```
    public boolean offer(E e) { // 默认头部增加
        return add(e);
    }
    public void push(E e) {
        addFirst(e);
    }
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
```

获取迭代器 可指定从**任何地方**迭代 同样也是快速返回失败的
```
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
```

链表转换为数组结构
```
    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }
```