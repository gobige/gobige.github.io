---
layout: post
title: '位运算实践'
subtitle: '位运算的一些应用实例'
date: 2019-08-30
categories: 高效
author: yates
cover: 'www.baidu.com'
tags:  技巧
---

## 位操作是对程序中二进制数的进行操作，在微处理器上，位运算比加减运算略快，通常位运算比乘除法运算要快很多。

### 基本概念

**判断奇偶**
判断int型变量a是奇数还是偶数 **a&1 = 0 偶数 a&1 = 1 奇数**

**取int型变量a的第k位**
a>>k&1

**将int型变量a的第k位清0**
a=a&~(1<<k)

**将int型变量a的第k位置1** 
a=a|(1<<k)

**int型变量循环左移k次**
a=a<<k|a>>16-k  

**int型变量a循环右移k次**
a=a>>k|a<<16-k  