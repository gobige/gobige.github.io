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
                if (arr[j] > waitSortVal) {
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
```java
public static int[] dichotomyInsertionSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int temp = arr[i];
        int left = 0;
        int right = i - 1;
        int mid = 0;
        while (left <= right) {
            mid = (left + right) / 2;
            if (temp < arr[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        for (int j = i - 1; j >= left; j--) {
            arr[j + 1] = arr[j];
        }
        if (left != i) {
            arr[left] = temp;
        }
    }


    return arr;
}
```

#### 希尔排序
希尔排序是选择一个小于数组大小n增量d，每隔d位的数字分为一组进行排序，然后继续选择一个小于之前增量的值继续分组排序，直到增量变为1

```java
public static int[] shellSort(int[] arr) {
    int incre = arr.length;
    while (true) {
        incre = incre / 2;

        // 分组
           for (int x = 0; x < incre; x++) {
            // 每组排序
            for (int i = x; i < arr.length; i = i + incre) {
                int waitSortVal = arr[i];

                // 每组进行插入排序
                for (int j = x; j < i; j = j + incre) {
                    if (arr[j] > waitSortVal) {
                        // 位移插入位置右边的数字
                        for (int k = i; k > j; k = k - incre) {
                            arr[k] = arr[k - incre];
                        }
                        arr[j] = waitSortVal;
                        break;
                    }
                }
            }
        }

        if (incre == 1) {
            break;
        }
    }

    return arr;
}
```

#### 选择排序
思路：每次从数组中取出最小的一个数字放到另一个数组中，同时删除获取的这个数字（优化方案：只在一个数组中进行该排序）

```java
    public static int[] selectSort(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            int minIndex = i;
            for (int j = i; j < arr.length; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            int temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }

        return arr;
    }
```

#### 冒泡排序
思路：在一个数组中，从第一个值开始，与下一位进行对比，如果大于下一位，则进行位置交换；然后继续下一位的对比，整个过程就像气泡从水底向水面游动，到水面时气泡时最大的,故此得名。
```java
public static int[] bubbleSort(int[] arr) {
    int temp;
    for (int i = 0; i < arr.length; i++) {
        for (int j = 0; j < arr.length - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }

    return arr;
}
```

#### 快速排序
先来了解几个概念
- 递归:程序调用自身的编程技巧称为递归。递归条件调用栈的概念和函数的关系,递归函数也用到了调用栈,
- 分而治之(D&C算法)工作原理:1找出简单的基线条件 2确定如何缩小问题规模,使其符合基线条件优雅的解决方法

思路：从数组中选择一个值作为基准数值，比该值大的分配到右边，比该值小的分配到左边，然后对左右两边的继续进行找基准数值，递归实现该方法
```java
public static void quickSort(int[] arrays, int start, int end) {
    // 如果每一轮基准排序结束后返回
	if(start > end){
		return;
	}

    int base = arrays[start];
	int i = start,j = end;
    int temp;
    // 保证调用不会内存泄露
    while (start < end) {
        // 从最右边开始查找如果比基准书小则结束查找，锁定位置
        while (base <= arrays[end] && start < end) {
            end--;
        }
        // 从最左边开始查找如果比基准数大则结束查找，锁定位置
        while (base >= arrays[start] && start < end) {
            start++;
        }
        // 交换比基准数大的和小的数值位置
        if (start < end) {
            temp = arrays[start];
            arrays[start] = arrays[end];
            arrays[end] = temp;
        }
    }

    // 基准数归位
    arrays[i] = arrays[start];
    arrays[start] = base;

    // 递归遍历
    quickSort(arrays,i,start - 1);
    quickSort(arrays,start + 1,j);
}
```

#### 堆排序
堆：堆是具有以下性质的完全二叉树：每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆；或者每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆。

思路：堆排序的基本思想是：将待排序序列构造成一个大顶堆，此时，整个序列的最大值就是堆顶的根节点。将其与末尾元素进行交换，此时末尾就为最大值。然后将剩余n-1个元素重新构造成一个堆，这样会得到n个元素的次小值。如此反复执行，便能得到一个有序序列了

```java

```