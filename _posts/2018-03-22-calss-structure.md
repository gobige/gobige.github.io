---
layout: post
title: 'class类文件结构'
subtitle: 'class类文件结构'
date: 2017-03-12
categories: jvm
author: yates
cover: ''
tags: jvm
---


# 前言
我们都知道java是一种基于jvm编译执行的语言，java文件编译成class文件，jvm只与Class文件这种特定的二进制文件格式所关联，任何能够编译成class文件结构的语言都可在jvm上运行如Scala，Groovy，Clojure等。

通常一个Class类文件都对应一个类和接口的定义信息，但是类和接口不一定定义在class文件里，也可以通过类加载器直接生成

class文件是由字节为基础单位的二进制流，各个数据项严格按照顺序紧凑排列在class文件中，顺序，数量，字节序都被严格按照位置存储

class文件格式如图

![此处输入图片的描述](http://www.muyibeyond.cn/img/2018-03-19-jvm/16.png)

**magic（魔数）**

class文件头4个字节为魔数，作为jvm进行身份识别的标志，为16进制的OxCAFEBABE