---
layout: post
title: '常用排序算法'
subtitle: '一些常用的排序算法'
date: 2017-09-08
categories: 算法
author: yates
cover: 'www.baidu.com'
tags: 排序算法
---

### 前言
排序可以分为两类，内排序和外排序。内排序：排序过程中，全部记录存放在内存中，称为内排序；外排序：排序过程中，记录存放需要用到外存，称为外排序。

常用排序的**空间复杂度**和**时间复杂度**


排序方法|时间复杂度（平均）|时间复杂度（最坏)|时间复杂度（最好)	|空间复杂度|稳定性|复杂性
:------:|:----------------:|:---------------:|:----------------:|:--------:|:----:|:----:|
直接插入排序|O(n2)	       |O(n2)            |	O(n)            |O(1)	   |稳定  |	简单
希尔排序|O(nlog2n)         |O(n2)            |O(n)	            |O(1)	   |不稳定|较复杂
直接选择排序|O(n2)         |O(n2)            |O(n2)             |O(1)      |不稳定|简单
堆排序	|O(nlog2n)|	O(nlog2n)|O(nlog2n)|	O(1)O(1)|	不稳定|	较复杂
冒泡排序|	O(n2)|	O(n2)|	O(n)|	O(1)|	稳定|	简单
快速排序|	O(nlog2n)|	O(n2)|	O(nlog2n)|	O(nlog2n)|	不稳定	|较复杂
归并排序|	O(nlog2n)|	O(nlog2n)|	O(nlog2n)|	O(n)|	稳定|	较复杂
基数排序|	O(d(n+r))|	O(d(n+r))|	O(d(n+r))|	O(n+r)|	稳定|	较复杂


#### 直接插入排序

思路：假设有n个值，第一次从n1作为待排序值，排序n1前面的值，第二次n2作为待排序值，排序n2前面的值;，第三次n3作为待排序值，排序n3前面的值；以此类推，代码如下：
```java
public static int[] directSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int waitSortVal = arr[i];

        for (int j = 0; j < i; j++) {
            if (arr[j] < waitSortVal) {
                for (int k = i; k > j; k--) {
                    arr[k] = arr[k-1];
                }
                arr[j] = waitSortVal;
                break;
            }
        }
    }

    return arr;
}
```

#### 二分法插入排序
思路：二分法插入排序的思想和直接插入一样，只是找到插入位置的方式不同，通过在已排完序中使用二分法查找插入位置，可以减少比较的次数。