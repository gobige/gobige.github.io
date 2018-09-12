---
layout: post
title: 'jvm垃圾收集器以及GC日志分析'
subtitle: 'jvm垃圾收集器以及GC日志分析'
date: 2017-03-08
categories: jvm
author: yates
cover: ''
tags: jvm
---

### 垃圾收集器
Serial：单线程收集器，GC时会采用StopWorld方式挂起所有执行线程，新生代采用复制，清除算法；老年代标记-整理算法 优点：单线程中简单高效

![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/4.png)

ParNew：Serial收集器的多线程版本
![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/5.png)

Parallel Scavenge收集器：新生代收集器，主要目的是达到一个控制的**吞吐量**（cpu运行用户代码/cpu总耗时），-XX:+UseAdaptiveSizePolicy开启GC自适应调节策略（根据最大吞吐量自动调节新生代，eden，survival的大小）

Serial Old：Serial控制器老年代版本，使用标记整理算法
![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/6.png)

Parallel Old：ParallelScavenge收集器老年代版本，使用标记整理算法
![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/7.png)


CMS
是一种目的是获取**最短回收停顿时间**为目标，使用标记-清除算法，多线程收集器，整个收集过程分为
1. 初始标记（仅仅标记GC Roots可达对象，速度很快，会stop the world）
2. 并发标记（进行GCRootTracing过程，用户线程可同时工作）
3. 重新标记（修正并发标记期间，用户程序继续运作导致标记产生变动的标记记录，一般比初试标记长，比并发标记短）
4. 并发清除
![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/8.png)

缺点：1 CMS收集器对CPU资源敏感，因为和用户线程同时工作，导致应用程序变慢，总吞吐量会降低，默认回收线程数（cpu数量+3)/4；2 无法处理浮动垃圾，3 标记-清除造成空间碎片，会造成提前full GC


G1收集器：是一款支持并行和并发，充分利用多CPU,多核优势，缩短STW停顿时间；具有采用分代收集；空间整合；可预测停顿等特点，在Java堆分配上面划分成多个大小相等的独立区域（region），新生代和老年代不再物理隔离，通过跟踪，分析Region里垃圾的价值大小，优先回收价值大的Region

垃圾回收整个过程分为
- 初始标记 （仅仅标记GC Roots可达对象，速度很快，会stop the world；标记可用region，使下一阶段用户线程可用）
- 并发标记（从GC Root开始对堆中对象进行可达性分析，找出存活对象，耗时较长）
- 最终标记（修正并发标记期间，用户程序继续运作导致标记产生变动的标记记录，需停顿线程，可并行执行）
- 筛选回收（对各个region回收价值和成本进行排序，根据用户所期望GC停顿时间指定回收计划）

![请输入图片地址](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/9.png)

### GC日志的理解
下面是一段GC日志

33.125：[GC[DefNew：3324K-＞152K（3712K），0.0025925secs]3324K-＞152K（11904K），0.0031680 secs] 
100.667：[FullGC[Tenured：0K-＞210K（10240K），0.0149142secs]4603K-＞210K（19456K），[Perm：2999K-＞2999K（21248K）]，0.0150007secs][Times：user=0.01 sys=0.00，real=0.02 secs]

GC发生时间：【GC停顿类型--【GC发生区域（名字跟收集器有关）--GC前内存->GC后内存大小（总大小），花费时间】GC前java堆大小->GC后java堆大小，花费时间】

- GC发生时间，代表从jvm启动以来经过的秒数
- GC停顿类型，分为minorygc和fullGC，而发生FullGC的原因可能是分配担保失败，也可能是程序中调用system.gc发生
- GC发生区域，一般会根据使用的收集器区分新生代，老年代区域
- 花费时间，收集器会给出具体时间数据，user-用户消耗cpu时间，sys-代表内核消耗cpu时间，real-操作开始到结束的墙钟时间（包含非运算使劲按等待消耗，如io，线程阻塞）