---
layout: post
title: 'jvm垃圾收集器以及GC日志分析'
subtitle: 'jvm垃圾收集器以及GC日志分析'
date: 2018-03-08
categories: jvm
author: yates																																									
cover: ''
tags: jvm
---

## 垃圾收集器

### Serial

单线程收集器，GC时会采用StopWorld方式挂起所有执行线程，新生代采用复制，清除算法；老年代标记-整理算法 优点：单线程中简单高效

![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/4.png)

### ParNew：

Serial收集器的多线程版本
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/5.png)

### Parallel Scavenge：

新生代收集器，主要目的是达到一个控制的**吞吐量**（cpu运行用户代码/cpu总耗时），-XX:+UseAdaptiveSizePolicy开启GC自适应调节策略（根据最大吞吐量自动调节新生代，eden，survival的大小）

### Serial Old：Serial:

控制器老年代版本，使用标记整理算法
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/6.png)

### Parallel Old：

ParallelScavenge收集器老年代版本，使用标记整理算法
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/7.png)


### CMS

老年代，方法区收集器，获取**最短回收停顿时间**为目标，超过老年代，方法区阙值进行GC,使用标记-清除算法。多线程收集器，整个收集过程分为

1. 初始标记（仅仅标记GC Roots可达对象，速度很快，会stop the world）
2. 并发标记（进行GCRootTracing过程，用户线程可同时工作）
3. 重新标记（修正并发标记期间，用户程序继续运作导致标记产生变动的标记记录，一般比初试标记长，比并发标记短）
4. 并发清除
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/8.png)

#### **缺点：**

1. CMS收集器对**CPU资源敏感**，因为和用户线程同时工作，导致**用户线程变慢**，总**吞吐量降低**，默认回收线程数（cpu数量+3)/4；
2. 无法处理**浮动垃圾**，
3. 标记-清除造成**空间碎片**，会造成**提前full GC**


### G1收集器：

新生代，老年代收集器

- 是一款支持**并行和并发**，充分利用多CPU,多核优势，**缩短STW**停顿时间；
- 具有采用**分代收集；空间整合；可预测停顿**等特点
- 在Java堆分配上面划分成多个大小相等的独立区域（**region**），新生代，存活区，老年代，大对象（Humongous大小超过region一半大小）**不再物理隔离**，-XX:G1HeapRegionSize配置
- 通过跟踪，分析Region里垃圾的价值大小，**优先回收价值大的Region**

**RSet**：个region初始化时，初始化RSet记录跟踪其他Region指向该Region中对象引用（每512KBregion划分为多个Card），垃圾回收时，很快的判断Region内对象是否存活
**CSets**：垃圾回收过程中会被回收的区域集合。

```java
-XX:+UseG1GC // 使用G1回收器
-XX:MaxGCPauseMillis=n  // 设置最大GC停顿时间
-XX:InitiatingHeapOccupancyPercent=n   // 并发GC周期时的堆内存占用百分比
-XX:NewRatio=n  // 新生代与老生代大小比例
-XX:ParallelGCThreads=n  // 垃圾收集器在并行阶段使用的线程数
-XX:ConcGCThreads=n  // 并发垃圾收集器使用的线程数量
```
设置应用的暂停时间：根据选择一个回收价值高的region进行回收。

G1比较适合内存稍大一点的应用

GC模式

- young GC，和eden region耗尽时触发
- mixed GC，越来越多对象晋升到old region时，避免堆内存被耗尽，回收整个eden region，一部分old region。 XX:InitiatingHeapOccupancyPercent配置老年代占整个堆百分比阙值
- full GC，对象分配过快，mixed来不及回收，老年代填满，触发full GC. serial old执行，长时间暂停

垃圾回收整个过程分为
- 初始标记 （仅仅标记GC Roots可达对象，速度很快，会stop the world；**标记可用region**，使下一阶段用户线程可用）
- 并发标记（从GC Root开始对堆中对象进行可达性分析，找出存活对象，耗时较长）
- 最终标记（修正并发标记期间，用户程序继续运作导致标记产生变动的标记记录，需**停顿线程**，可**并行**执行）
- 筛选回收（对各个region**回收价值和成本进行排序**，根据**用户期望GC停顿时间**指定回收计划）

![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/9.png)



- 服务端多核CPU、JVM内存占用较大的应用（至少大于4G）
- 应用在运行过程中会产生大量内存碎片、需要经常压缩空间
- 想要更可控、可预期的GC停顿周期；防止高并发下应用雪崩现象

### ZGC收集器：
可伸缩的、低延迟的垃圾收集器；停顿时间不会随堆增大而增大，停顿时间不会超过10ms，支持几百M，最大4T堆大小

- Concurent：短暂的STW，大部分时间和应用程序并发执行，标记和移动最慢
- Region-based：以page为单位进行对象分配和回收
- Compacting：每次GC，都会对page进行压缩操作，避免碎片化问题
- NUMA-aware：创建对象根据当前线程在哪个CPU执行，优先靠近CPU内存进行分配
- Using colored poinrers，ZGC标记标记对象的指针而不是在对象头进行垃圾回收标记
- Using load barriers：因为标记和移动，为了应用程序获取对象时对的，经过一个读屏障，保证执行GC时，读取数据正确性

GC触发策略

任有空闲内存情况下

- rule_timer:周期性GC，XX:ZCollectionInterval= 1,每隔一段时间ZGC
- rule_warmup:JVM启动后，一直没GC，在10,20,30个百分点阙值时，分别触发一次GC，收集GC数据为后面GC提供条件依据
- rule_allocation_rate：根据对象分配速率（估计很快耗尽内存）决定是否GC。
- rule_proactive：允许吞吐量下降情况下，积极主动GC
	- 自从上次GC后，堆使用量至少涨10%
	- 自从上次GC后，过去5分钟没有GC

没有空闲内存情况下:stw,回收一部分内存，让应用程序继续执行

### GC日志设置

```java
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log 日志文件的输出路径
```

### GC日志的理解
下面是一段GC日志

33.125：[GC[DefNew：3324K-＞152K（3712K），0.0025925secs]3324K-＞152K（11904K），0.0031680 secs] 
100.667：[FullGC[Tenured：0K-＞210K（10240K），0.0149142secs]4603K-＞210K（19456K），[Perm：2999K-＞2999K（21248K）]，0.0150007secs][Times：user=0.01 sys=0.00，real=0.02 secs]

GC发生时间：【GC停顿类型--【GC发生区域（名字跟收集器有关）--GC前内存->GC后内存大小（总大小），花费时间】GC前java堆大小->GC后java堆大小，花费时间】

- GC发生时间，代表从jvm启动以来经过的秒数
- GC停顿类型，分为minorygc和fullGC，而发生FullGC的原因可能是分配担保失败，也可能是程序中调用system.gc发生
- GC发生区域，一般会根据使用的收集器区分新生代，老年代区域
- 花费时间，收集器会给出具体时间数据，user-用户消耗cpu时间，sys-代表内核消耗cpu时间，real-操作开始到结束的墙钟时间（包含非运算使劲按等待消耗，如io，线程阻塞）

### G1 GC日志解读

```java
[GC pause (G1 Evacuation Pause) (young), 0.0077244 secs]  // 垃圾回收类型，花费时间
   [Parallel Time: 4.1 ms, GC Workers: 8] // 并行收集 STW花费时间；垃圾收集开启线程数，CPU小于8，最大设置为8，其他 number * 5/8
      [GC Worker Start (ms): Min: 470282.4, Avg: 470282.6, Max: 470283.1, Diff: 0.6] // 第一个（min）/最后一个（max）垃圾线程工作开始时间
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.6, Max: 2.2, Diff: 2.2, Sum: 4.7] // 扫描GCroot集合时间
      [Update RS (ms): Min: 0.0, Avg: 0.8, Max: 1.2, Diff: 1.2, Sum: 6.3] // 垃圾收集线程处理 垃圾收集 前没有处理好的日志缓冲区
         [Processed Buffers: Min: 0, Avg: 5.4, Max: 11, Diff: 11, Sum: 43] // 处理多少个日志缓冲区
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1] // 扫描每个新生代分区的RSet，找出有多少指向当前分区的引用来自CSet。
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.2, Max: 1.3, Diff: 1.3, Sum: 1.6] // 扫描代码中的root节点（局部变量）花费的时间
      [Object Copy (ms): Min: 1.3, Avg: 2.2, Max: 2.6, Diff: 1.3, Sum: 17.5] // 将当前分区中存活的对象拷贝到新的分区
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0] // 当一个垃圾收集线程完成任务时，它就会进入一个临界区，并尝试帮助其他垃圾线程完成任务,尝试terminate（min），开始terminate(max)
         [Termination Attempts: Min: 1, Avg: 6.4, Max: 10, Diff: 9, Sum: 51] // 获取其他线程任务次数
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.2] // 垃圾收集线程在完成其他任务的时间
      [GC Worker Total (ms): Min: 3.3, Avg: 3.8, Max: 3.9, Diff: 0.7, Sum: 30.4] // 每个垃圾收集线程的最小、最大、平均、差值和总共时间
      [GC Worker End (ms): Min: 470286.3, Avg: 470286.4, Max: 470286.4, Diff: 0.0] // 最早/最晚结束垃圾回收线程时间
   [Code Root Fixup: 0.1 ms] // 释放用于管理并行垃圾收集活动的数据结构
   [Code Root Purge: 0.0 ms] // 清理更多的数据结构
   [Clear CT: 0.9 ms] // 清理card table
   [Other: 2.7 ms]
      [Choose CSet: 0.0 ms] // 选择要进行回收的分区放入CSet
      [Ref Proc: 1.5 ms] // 处理Java中的各种引用
      [Ref Enq: 0.0 ms] // 遍历所有的引用，将不能回收的放入pending列表
      [Redirty Cards: 1.0 ms] // 回收过程中被修改的card将会被重置为dirty
      [Humongous Register: 0.0 ms]// 巨型对象可以在新生代收集的时候被回收——通过G1ReclaimDeadHumongousObjectsAtYoungGC 设置，默认为true
      [Humongous Reclaim: 0.0 ms] // 确保巨型对象可以被回收、释放该巨型对象所占的分区，重置分区类型，并将分区还到free列表，并且更新空闲空间大小。
      [Free CSet: 0.1 ms] // 将要释放的分区还回到free列表
   [Eden: 105.0M(105.0M)->0.0B(105.0M) Survivors: 9216.0K->9216.0K Heap: 220.4M(256.0M)->115.4M(256.0M)]
 [Times: user=0.00 sys=0.00, real=0.01 secs] 
```

### CPU过高排查步骤

- 使用top命令查看进程的系统使用指标，找出cpu消耗高的进程id
- 使用top -Hp {pid}查看哪些线程CPU过高，
- 通过jstack查看线程堆栈信息
- 看出具体使用CPU高的代码位置；若显示VM-tread则表示垃圾回收线程，很有可能是Full GC频繁导致
- 使用stat 查看进程GC情况是否是Full GC 频繁；dump除内存，hat分析具体触发Full GC代码位置

Major GC是清理老年代，Full GC是清理整个堆空间

### GC调优

引起FULL GC原因

- 老年代不够用了，大对象，对象晋升，达到阈值
- 永久代元空间不够用了，达到阈值扩容
- system.gc

**减少线程数量**
减少线程数量有利于减少GC roots Scanning，降低单次young gc时间

**降低minor GC 频率，增加年轻代空间**

单次Minor GC由两部分组成T1（扫描新生代），T2（复制存活对象），如果堆内存较多**长期存活对象**，那么**Eden区越大**，Minor GC**时间越长**；如果**短对象多**，那么**Eden区越大**，MinorGC时间不会显著增加；所以，**单次Minor GC时间更多取决于GC后存活对象数量**，而非Eden区大小

**减少对象生成**

尽量使用小对象，方法内和线程分配对象，利用JIT优化对象栈上分配；使用对象池

**降低Full GC频率**

- 减少创建大对象，一次性查询很多字段的对象，大对象直接创建在老年代。
- 增大堆内存空间
- 选择合适GC回收器

Ergonomics就是负责自动的调解gc暂停时间和吞吐量之间的平衡，开启了UseAdaptiveSizePolicy，jvm自己进行自适应调整引发的full gc

TLAB 分配

TLAB，全称Thread Local Allocation Buffer, 即：线程本地分配缓存。
这是一块线程专用的内存分配区域。TLAB占用的是eden区的空间。在TLAB启用的情况下（默认开启），JVM会为每一个线程在eden区分配一块TLAB区域而不用线程同步（大对象无法进行TLAB分配）


**设置Eden，Survivor区比例**

```java
java -XX:+PrintFlagsFinal -version | grep HeapSize // 查看JVM堆内存分配
-XX:+DoEscapeAnalysis  // 开启逃逸分析 
-XX:-DoEscapeAnalysis  //  关闭逃逸分析
-XX:-UseTLAB	 // 关闭线程本地分配缓存区
-XX:+EliminateAllocations	// 开启标量替换
-XX:-EliminateAllocations   // 关闭标量替换
	


-XX:+HeapDumpBeforeFullGC -XX:+HeapDumpAfterFullGC // full gc前，full gc后做heap dump
-XX:PretenureSizeThreshold  // 大对象直接分配老年代阈值
-XX:+UseAdaptiveSizePolicy // JVM动态调整Java堆中各个区域大小以及进入老年代年龄，-XX：NewRatio和-XX：SurvivorRatio会失效，JDK8默认开启
```

**有时候通过工具只能猜测到可能是某些地方出现了问题，而实际排查则要结合源码做具体分析。**
