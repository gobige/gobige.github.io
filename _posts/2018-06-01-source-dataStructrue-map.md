---
layout: post
title: 'java看源码系列-map集合'
subtitle: 'openjdk HashMap LinkedHashMap TreeMap hashTable的实现'
date: 2018-06-01
categories: 个人博客
author: yates
cover: 'www.baidu.com'
tags: source
---

**map接口定义了所需实现方法**
- int size();
- boolean isEmpty();
- boolean containsKey(Object key);
- boolean containsValue(Object value);
- V get(Object key);
- V put(K key, V value);
- V remove(Object key);
- void putAll(Map<? extends K, ? extends V> m);
- void clear();   
- boolean equals(Object o);
- int hashCode();

**entry接口定义map集合元素entry所需实现方法**
- K getKey();
- V getValue();
- V setValue(V value);
- boolean equals(Object o);
- int hashCode();
- comparingByKey 实现了自身以key作为对比规则的对比器
- comparingByValue 实现了自身以value作为对比规则的对比器
- comparingByKey(Comparator<? super K> cmp)  实现了使用指定比较器比较key的比较器
- comparingByValue(Comparator<? super V> cmp)  实现了使用指定比较器比较value的比较器
- boolean equals(Object o);

**AbstractMap抽象类**
部分的实现,重写了map接口的方法 hashCode equals removeAll

**hashMap类**

```java
// 拥有一个Node类型对象数组
transient Node<K,V>[] table;
// entry类型set集合
transient Set<Map.Entry<K,V>> entrySet;
// map集合大小
transient int size;
// map对象操作次数
transient int modCount;
// 下一次扩容容量
int threshold;
// 容量扩容增长因子
final float loadFactor; // 当查询操作较为频繁时，我们可以适当地减少加载因子；如果对内存利用率要求比较高，我可以适当的增加加载因子。
```

Node静态内部类，实现了entry接口,具有下列变量

```java
 static class Node<K,V> implements Map.Entry<K,V> 
 
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
```

对key进行hash计算
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

构造函数 默认增长因子为0.75 默认初始化容量大小 如不指定 为0
```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

获取给定数的大于或等于2的整数次方数
```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

获取map大小
```java
public int size() {
    return size;
}
```

putval方法四个入参： 需要查找的key的**hash值**；存入**key对象**；**存入value对象**；talbe数组中node元素以**key为标识的的value对象**是否只能**赋值一次**；这个boolean类型的参数相当的与java中的**final**修饰变量；

```java
public V put(K key, V value) {
                    ①
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
                    ②
        n = (tab = resize()).length;
                        ③
    if ((p = tab[i = (n - 1) & hash]) == null)
                        ④
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
                    ⑤
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                ⑥
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
                ⑦
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```
插入一个node节点:

① 先将key值**hash计算（查询会很快）**， 

② 如果table为null或者数组没有一个node节点则调用resize进行扩容(resize方法扩容会**重新计算集合中元素的位置**，重新插入，所以我们在使用hashmap作为集合进行数据的封装时候，最好是**指定集合的初始化大小**)

③ **(n - 1) & hash**请记住这个公式，通过hash值和整个table数组的大小n使用**与运算**快速的计算出插入的node应该位于该table数组中的位置 

④ 如果计算出来的位置**没有指针引用到任何对象**，那么调用newNode方法直接创建node节点存入该位置，这时我们会发现（**hashMap的key，value都可为null,key为null有且只有一个node对象**）

⑤ 如果该位置已经有node对象了，通过**key是否是同一个对象或者key对象自身实现的equal方法判定是否相等**判断存入的是node对象**是否应该存入该位置** 

⑥ 如果存入的key和该位置已存在key相等，那么**只更新该位置node对象的value（key对象在hashmap中唯一）** 

⑦ 如果不相等则获取**该位置node对象指向的下一个节点**，判断下一个节点的**是否为null**，如果为null则将创建节点赋值key，value存入，如果不为null，则继续判断是否key值相等，相等则更新该节点value，不相等则继续查找下一节点，理论节点数量是可无穷存储的**（由此可以看出hashmap是一个数组链表的存储结构）**，但是如果链表结构过长，hashmap会使用**treeifyBin**方法将链表中的node对象**替换为treeNode对象**进行存储

treeNode类结构（**就是一个红黑树的结构**）
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
```

treeifyBin方法
```java
 final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

根据key获取map中的node对象或treeNode对象
```java
// 先将给定key进行hash计算，得到hash值
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
// 是否包含指定key的node元素
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
    // 通过hash值
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}    
```

根据key删除node或trenode节点：
1先查找**table数组中node节点**是否有匹配，如果未找到在查找对应hash值位置下节点**下一节点**，如此**循环直到找到或到达链表尽头**，将找到的要**删除node节点的下一个节点**赋值给**当前table数组中位置指针**或**上一个节点的next指针**
```java
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
    
        final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

hashMap清空  赋值table数组**所有元素为null**
```java
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```

hashMap查找是否包含value对象,循环数组table，**嵌套循环node节点**
```java
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```

hashMap获取所有key的set集合(**可以看出来keySet类是hashMap的一个内部类，单我们使用它遍历，迭代的时候，其本质还是还是从hashMap的node对象数组进行操作**)
```java
 public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

hashMap获取所有value的collelction集合(**可以看出来values类也是hashMap的一个内部类，和keyset一样**)
```java
  public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }

    final class Values extends AbstractCollection<V> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<V> iterator()     { return new ValueIterator(); }
        public final boolean contains(Object o) { return containsValue(o); }
        public final Spliterator<V> spliterator() {
            return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super V> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.value);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```


hashMap获取由key和value组合形成的entry对象set集合(**可以看出来keySet类是hashMap的一个内部类，我们可以看到entry对象里的contains，remove,foreach,迭代本质还是从hashMap的node对象数组链表进行操作**)
keyset，values和entryset遍历的时候也是会快速失败的，所以hashMap也是线程不安全的
```java
   public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
    }

    final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Map.Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

获取**指定key**的node元素，如果不存在返回默认value
```java
    @Override
    public V getOrDefault(Object key, V defaultValue) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
    }
```

插入键，值如果不存在（key值对应**node不存在**或**node存在，但是node的value对象为null**）
```java
    public V putIfAbsent(K key, V value) {
        return putVal(hash(key), key, value, true, true);
    }
```

删除指定key，而且value为指定的node节点 
```java
    @Override
    public boolean remove(Object key, Object value) {
        return removeNode(hash(key), key, value, true, true) != null;
    }
```

替换指定key的node节点的value值
```java
    @Override
    public V replace(K key, V value) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) != null) {
            V oldValue = e.value;
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
        return null;
    }
```

替换指定key，而且value为指定的node节点的value值
```java
    @Override
    public boolean replace(K key, V oldValue, V newValue) {
        Node<K,V> e; V v;
        if ((e = getNode(hash(key), key)) != null &&
            ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
            e.value = newValue;
            afterNodeAccess(e);
            return true;
        }
        return false;
    }
```

computeIfAbsent 待解析
computeIfPresent 待解析
compute 待解析
merge 待解析
forEach(BiConsumer<? super K, ? super V> action) 待解析 
replaceAll(BiFunction<? super K, ? super V, ? extends V> function) 待解析
HashIterator
KeyIterator
ValueIterator
EntryIterator
HashMapSpliterator
KeySpliterator
ValueSpliterator
EntrySpliterator


![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/performance/3.png)

**LinkedHashMap类**
继承HashMap,下面是linkedHashMap的结构(可以看出来LinkedHashMap完全是一个链表的结构，而且是双向链表，**查询的时候会比hashMap慢**)

```java
    // 实现了一个新的entry类，继承hashMap的node类
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

定义了头节点和尾节点
```java
    transient LinkedHashMap.Entry<K,V> head;
    
    transient LinkedHashMap.Entry<K,V> tail;
```


定义了该链表结构的迭代排序方式  **true：访问顺序** **false：插入顺序**

```java
    final boolean accessOrder;
```

链接指定节点到该链表尾部
```java
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

链接指定的src替换链表中指定dst的位置，设置为私有方法 不可单独用，会造成**闭环**
```java
    private void transferLinks(LinkedHashMap.Entry<K,V> src,
                               LinkedHashMap.Entry<K,V> dst) {
        LinkedHashMap.Entry<K,V> b = dst.before = src.before;
        LinkedHashMap.Entry<K,V> a = dst.after = src.after;
        if (b == null)
            head = dst;
        else
            b.after = dst;
        if (a == null)
            tail = dst;
        else
            a.before = dst;
    }
```

创建一个新的entry节点，自动插入到尾节点
```java
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
```

替换p节点尾next节点
```java
    Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        LinkedHashMap.Entry<K,V> t =
            new LinkedHashMap.Entry<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }

```

生成一个新的树形节点和替换一个树形节点
```java
    TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
        TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
        linkNodeLast(p);
        return p;
    }

    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        TreeNode<K,V> t = new TreeNode<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }
```

删除某个节点后（**配合热moveNode方法使用**删除e节点对前后节点的关联，对e节点前后关联的节点进行新的关联）
```java
 void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }
```

增加某个节点后
```java
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```

访问模式访问节点后操作（**配合访问规则下的get（key）方法使用**）

```java
     void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
}
```

linkedHahMap构造函数 默认采用**插入规则**
```java
     */
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
```

linkedHashMap链表是否包含指定value值的节点
```java
    public boolean containsValue(Object value) {
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }
```

获取指定key的node节点value
```java
   public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

清空linkedHashMap
```java
    public void clear() {
        super.clear();
        head = tail = null;
    }
```

获取linkedHashMap的所有key的set集合（**其本质还是遍历linkedhashmap的entry对象获取key**）
```java
public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new LinkedKeySet();
            keySet = ks;
        }
        return ks;
    }

    final class LinkedKeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<K> iterator() {
            return new LinkedKeyIterator();
        }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator()  {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED |
                                            Spliterator.DISTINCT);
        }
        public final void forEach(Consumer<? super K> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e.key);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
```


获取linkedHashMap的所有key的collection集合（**其本质还是遍历linkedhashmap的entry对象获取value**）
```java
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new LinkedValues();
            values = vs;
        }
        return vs;
    }

    final class LinkedValues extends AbstractCollection<V> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<V> iterator() {
            return new LinkedValueIterator();
        }
        public final boolean contains(Object o) { return containsValue(o); }
        public final Spliterator<V> spliterator() {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED);
        }
        public final void forEach(Consumer<? super V> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e.value);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
```
   
   
linkedHashMap获取所有entry集合
```java
public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new LinkedEntrySet()) : es;
    }

    final class LinkedEntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new LinkedEntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Map.Entry<K,V>> spliterator() {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED |
                                            Spliterator.DISTINCT);
        }
        public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
```
LinkedHashIterator
LinkedKeyIterator
LinkedValueIterator
LinkedEntryIterator

**Hashtable**
hashTable就是一个**对元素操作加了锁的hashmap** 值得注意的是hashtable是继承的Dictionary接口

hashTable变量定义
```java
private transient Entry<?,?>[] table; 
private transient int count;
private int threshold;
private float loadFactor;
private transient int modCount = 0;
private transient volatile Set<K> keySet;
private transient volatile Set<Map.Entry<K,V>> entrySet;
private transient volatile Collection<V> values;
```

hashTable的hash算法
```java
(hash & 0x7FFFFFFF) % tab.length
```

获取entry对象中key和values的枚举集合（）
```java
public synchronized Enumeration<K> keys() {
    return this.<K>getEnumeration(KEYS);
}
public synchronized Enumeration<V> elements() {
    return this.<V>getEnumeration(VALUES);
}

// keys values等常量的定义
private static final int KEYS = 0;
private static final int VALUES = 1;
private static final int ENTRIES = 2;
private <T> Enumeration<T> getEnumeration(int type) {
    if (count == 0) {
        return Collections.emptyEnumeration();
    } else {
        return new Enumerator<>(type, false);
    }
}
```

插入一个键值对（可以看出hashtable **不予许插入的value和key为null**）

```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}

private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```

hashTable获取所有key的set集合,生成的set集合也是线程安全的
```java
 */
public Set<K> keySet() {
    if (keySet == null)
        keySet = Collections.synchronizedSet(new KeySet(), this);
    return keySet;
}
```

hashTable获取所有values的set集合,生成的collection集合也是线程安全的
```java
public Collection<V> values() {
    if (values==null)
        values = Collections.synchronizedCollection(new ValueCollection(),
                                                    this);
    return values;
}
```

hashTable的equals方法 先判断比较集合的类型，然后是集合的大小，然后集合中元素的位置和值是否一致
```java
public synchronized boolean equals(Object o) {
    if (o == this)
        return true;

    if (!(o instanceof Map))
        return false;
    Map<?,?> t = (Map<?,?>) o;
    if (t.size() != size())
        return false;

    try {
        Iterator<Map.Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext()) {
            Map.Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            if (value == null) {
                if (!(t.get(key)==null && t.containsKey(key)))
                    return false;
            } else {
                if (!value.equals(t.get(key)))
                    return false;
            }
        }
    } catch (ClassCastException unused)   {
        return false;
    } catch (NullPointerException unused) {
        return false;
    }

    return true;
}
```

hashTable的EntrySet，ValueCollection对象也只是hashTable的一个封装，里面的对元素进行操作的方法本质还是在**hashtable的数组链表结构**中进行操作

remove  putIfAbsent replaceAll replace computeIfPresent merge等方法的逻辑基本和hashMap大同小异了


**TreeMap**
treeMap的定义的变量（可以看出来定义了**排序的比较器**）
```java
private final Comparator<? super K> comparator;
private transient Entry<K,V> root;
private transient int size = 0;
private transient int modCount = 0;
```

判断是否包含指定value的entry元素 
```java
public boolean containsValue(Object value) {
    for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e))
        if (valEquals(value, e.value))
            return true;
    return false;
}

final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}

static final boolean valEquals(Object o1, Object o2) {
    return (o1==null ? o2==null : o1.equals(o2));
}

static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

获取指定key的元素(如果treemap有指定**比较器**，则用给定的比较器进行树的遍历，直到找到相应key的节点对象或无法找到指定key的节点对象，我们指定二叉树的平衡性决定了树的查找的速度，而treeMap使用了**红黑树**)
```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
```

插入指定的键值对(如果插入集合中无根节点，则将该键值对作为根元素插入，如果有根节点，则对比是否已有指定key的元素（则将根节点与插入节点的key通过比较器进行对比），有则覆盖，无则在合适的节点上增加该节点，key看出，treemap结构是典型的**二叉树结构**，**不允许key为null且唯一**，)
```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

final int compare(Object k1, Object k2) {
    return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
        : comparator.compare((K)k1, (K)k2);
}
```

插入指定map集合中所有元素
```java
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map);
}
```

删除指定key的元素
```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
    
private void deleteEntry(Entry<K,V> p) {
modCount++;
size--;

// If strictly internal, copy successor's element to p and then make p
// point to successor.
if (p.left != null && p.right != null) {
    Entry<K,V> s = successor(p);
    p.key = s.key;
    p.value = s.value;
    p = s;
} // p has 2 children

// Start fixup at replacement node, if it exists.
Entry<K,V> replacement = (p.left != null ? p.left : p.right);

if (replacement != null) {
    // Link replacement to parent
    replacement.parent = p.parent;
    if (p.parent == null)
        root = replacement;
    else if (p == p.parent.left)
        p.parent.left  = replacement;
    else
        p.parent.right = replacement;

    // Null out links so they are OK to use by fixAfterDeletion.
    p.left = p.right = p.parent = null;

    // Fix replacement
    if (p.color == BLACK)
        fixAfterDeletion(replacement);
} else if (p.parent == null) { // return if we are the only node.
    root = null;
} else { //  No children. Use self as phantom replacement and unlink.
    if (p.color == BLACK)
        fixAfterDeletion(p);

    if (p.parent != null) {
        if (p == p.parent.left)
            p.parent.left = null;
        else if (p == p.parent.right)
            p.parent.right = null;
        p.parent = null;
    }
}
}
```
 