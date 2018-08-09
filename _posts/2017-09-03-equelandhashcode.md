---
layout: post
title: 'equels和hashCode'
subtitle: 'equels和hashCode'
date: 2017-09-03
categories: java底层
author: yates
cover: 'http://cctv.com'
tags: jdk
---
### 前言

java的基类Object提供了equals和hashCode方法，前者判断连个对象是否相等，后者用于计算对象的哈希码
## equals
object类中equals中是这样写的
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
该实现采用了 区分度最高的算法，只要连个对象不是同一个对象就返回false

equals方法可以重写，但是要遵守**约定**

- 自反性：x.equals(x)必须返回true
- 对称性：x.equals(y)与y.equals(x)返回值相等
- 传递性：x.equals(y) 为true，y.equals(z)为ture，那么x.equals(z)也必须为ture
- 一致性：如果x.equals(y)中使用信息没有改变，那么y.equals(x)始终不变
- 非null：x不是null，y为null，那么x.equals(y)必须为false

## hashCode
object类中hashCode中是这样写的
```java
public native int hashCode();
```
可以看出是本地方法，实际上是将该对象在**内存中的地址作为哈希码**返回，可以保证不同对象哈希码不一样

hashCode重写也有**注意事项**

- hashCode在hash表中起作用
- 重写后的hashCode方法不能太简单，否则会导致哈希冲突较多；同样也不能太复杂，否则会导致计算影响性能
- 如果对象在equals中使用的信息不变，那么hashcode始终不变
- 如果两个对象使用equals方法判断相等，那么hashcode也应该相等
- 反之如果两个对象quals方法判断不相等，那么hashcode也必须应该不相等

hashMap和hashSet的实现也是遵循上面的规则的，当哈希表中插入对象时，通过获取对象哈希码直接定位，如果该位置没有对象，则将对象插入到该位置，如果该位置已经有对象，则通过equals判断是否是同一个对象，如果是则不插入，如果不是则将对象加入到链表中


## integer中的equals和hashCode
```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
public static int hashCode(int value) {
    return value;
}
```

## String中的equals和hashCode
```java
public boolean equals(Object var1) {
    if (this == var1) {
        return true;
    } else {
        if (var1 instanceof String) {
            String var2 = (String)var1;
            int var3 = this.value.length;
            if (var3 == var2.value.length) {
                char[] var4 = this.value;
                char[] var5 = var2.value;

                for(int var6 = 0; var3-- != 0; ++var6) {
                    if (var4[var6] != var5[var6]) {
                        return false;
                    }
                }

                return true;
            }
        }

        return false;
    }
}

public int hashCode() {
    int var1 = this.hash;
    if (var1 == 0 && this.value.length > 0) {
        char[] var2 = this.value;

        for(int var3 = 0; var3 < this.value.length; ++var3) {
            var1 = 31 * var1 + var2[var3];
        }

        this.hash = var1;
    }

    return var1;
}
```

通过代码可以看出

- String中的hashCode是缓存了的，一切为了性能
- String的hash计算公式是**h = 31 * h + val[i]**,那么问题来了，为什么是31呢？
      
1. 使用质数计算哈希码，由于特性，与其他数字相乘后结果唯一概率更大，哈希冲突概率更小
2. 质数越大，哈希冲突概率越小，但是运算速度越慢，选择31是实践后择中的选择
3. 我们都知道位运算快于其他运算，jvm会对31进行优化 31 * i == （i << 5） - i

- String的eauals相等条件是同为String类型，长度相同，且数组中字符完全相同，不要求二者是同一个对象
