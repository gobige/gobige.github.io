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

==、equals()：是用于比较对象以及。它们的存在解决了两个根本问题：如何判断两个对象是否“相同”；hashCode() ：支持高效数据结构（如哈希表）的核心机制

## ==（运算符）

- 作用：用于比较。
    - 基本数据类型（如 int, double）：比较数值是否相等。
    - 引用数据类型（对象），比较它们在内存中的地址是否相同（即它们是不是指向堆内存中的同一个对象）。

特点：速度极快，是底层最直接的比较方式。


java的基类Object提供了equals和hashCode方法，前者判断两个对象是否相等，后者用于计算对象的哈希码

## equals

- 作用：判断两个对象是否“相等”
- **默认行为**：未被重写时，Object 的 equals() 方法底层直接使用 == 实现，也就是比较**内存地址**。
- **重写目的**：Java 中的许多类（如 String、Integer、LocalDate 等）重写了该方法，用于比较对象的**逻辑内容**

equals方法**重写规约**

- 自反性：x.equals(x)必须返回true
- 对称性：x.equals(y)与y.equals(x)返回值相等
- 传递性：x.equals(y) 为true，y.equals(z)为ture，那么x.equals(z)也必须为ture
- 一致性：如果x.equals(y)中使用信息没有改变，那么y.equals(x)始终不变
- 非null：x不是null，y为null，那么x.equals(y)必须为false

## hashCode

- 作用：返回该对象的**哈希码（一个 int 类型的整数）**。
- **默认行为**：根据对象的**内存地址计算**得出。
- 主要**用途**：配合基于哈希机制的集合类。当向这些集合中添加或查找元素时，通过 hashCode() 计算出对象的存储位置（桶位），从而实现 $O(1)$ 级别的**高效查找**。

hashCode重写**注意事项**

- 重写后的hashCode方法不能太简单，否则会导致哈希冲突较多；同样也不能太复杂，否则会导致计算影响性能
- 如果对象在equals中使用的信息不变，那么hashcode始终不变
- 如果**两个对象使用equals方法判断相等，那么hashcode也应该相等**

-------------------

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
