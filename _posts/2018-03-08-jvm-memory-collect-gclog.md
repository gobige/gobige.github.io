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


G1 GC 的核心特点
- 停顿时间可预测（MaxGCPauseMillis）：你可以给它设定一个期望值（比如 -XX:MaxGCPauseMillis=200，表示单次 GC 停顿尽量不超过 200ms）。G1 会根据历史数据动态计算，在满足该停顿时间的前提下，尽可能多地回收垃圾。
- 面向全局的内存布局：彻底抛弃了传统收集器（如 CMS/Parallel）将堆物理划分为固定大小“年轻代”和“老年代”的做法。
- 全内存碎片整理：由于使用“复制算法”进行 Region 间的对象移动，G1 回收后不会产生内存碎片。

2. G1 的内存布局
- G1 将整个 Java 堆物理上划分为约 2048 个大小相等、连续的独立区域 —— Region（每个 Region 大小在 1MB 到 32MB 之间）。
	- 每个 Region 的身份是动态的，它可能这一刻是年轻代，下一刻被回收后又变成了老年代：
		- E (Eden)：年轻代伊甸园区。
		- S (Survivor)：年轻代存活区。
		- O (Old)：老年代区。
		- H (Humongous)：巨型对象区。如果一个对象的大小超过了 Region 的 50%，就会被直接放到 H 区（防止大对象在 E 和 S 区之间来回复制产生高额开销）。

3. G1 GC 的垃圾回收原理
G1 的回收核心思想是 “垃圾优先（Garbage-First）”：它会跟踪每个 Region 里的垃圾价值（回收能释放多少内存、需要消耗多少时间），并在后台维护一个优先级列表，每次只优先回收价值最大的那些 Region。

它的回收主要分为两套并发的机制：
- 机制 A：年轻代回收（Young GC）
	- 当 Eden 区满了之后，G1 会触发 Young GC。它会把所有 Eden 和 Survivor 区的活对象复制到新的 Survivor 区或者老年代区。这个过程是 Stop-The-World (STW) 的，但因为年轻代对象死得快，且采用多线程并行复制，速度极快。
- 机制 B：混合回收（Mixed GC）—— G1 的精髓
	- 当老年代的内存占用达到一定阈值时（默认 45%），会触发 Mixed GC。它不仅回收年轻代，还会回收部分老年代的 Region。其工作原理分为四个阶段：
	- 初始标记 (Initial Mark) 【STW】：只标记一下 GC Roots 能直接关联到的对象。这个阶段会借道 Young GC 同步完成，所以停顿极短。
	- 并发标记 (Concurrent Mark)：从 GC Roots 开始对堆中对象进行可达性分析，找出存活对象。这个阶段是与应用程序并发运行的，耗时最长但不会卡顿用户线程。
	- 最终标记 (Remark) 【STW】：由于阶段 2 用户线程还在运行，会产生一些对象引用的变动。G1 采用 SATB（原始快照） 算法，短暂挂起线程，快速修正并记录这些变动。
	- 筛选回收 (Cleanup / Evacuation) 【STW】：G1 对各个 Region 的回收价值和成本进行排序，根据用户设定的期望停顿时间，计算出这次应该回收哪些 Region。然后将选中的 Region 里的活对象复制（搬家）到空闲的全新 Region 中，直接清空旧 Region。

**G1 内部的“黑科技”：Remembered Set (RSet)**
Region 这么多，当回收年轻代 Region 时，万一老年代对象引用了年轻代对象怎么办？难道要扫描整个老年代吗？
G1 为每个 Region 都配了一个 RSet（记忆集）。谁引用了我，我就在我的 RSet 里记录下来。这样在 GC 时，只需扫描目标 Region 的 RSet 就能知道跨代引用，避免了全堆扫描。
 
```java
-XX:+UseG1GC // 使用G1回收器
-XX:MaxGCPauseMillis=n  // 设置最大GC停顿时间
-XX:InitiatingHeapOccupancyPercent=n   // 并发GC周期时的堆内存占用百分比
-XX:NewRatio=n  // 新生代与老生代大小比例
-XX:ParallelGCThreads=n  // 垃圾收集器在并行阶段使用的线程数
-XX:ConcGCThreads=n  // 并发垃圾收集器使用的线程数量
```
设置应用的暂停时间：根据选择一个回收价值高的region进行回收。

G1比较适合更可控、可预期的GC停顿周期；防止高并发下应用雪崩现象，内存稍大一点的应用

![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/9.png)


### ZGC (Z Garbage Collector)：
如果你的系统是超大堆（几十G甚至上百G），且对延迟（Latency）极度敏感（比如电商大促、量化交易、金融风控），G1 在 Mixed GC 时的停顿还是可能会达到几百毫秒。这时候最先进的 ZGC 就登场了。

1. ZGC 的恐怖特点：
- 停顿时间不超过 10ms（甚至通常在 1ms 以内）。
- 停顿时间不随着堆大小的增加而增加。不管你是 10GB 还是 16TB 的堆，停顿时间都差不多。

2. ZGC 的工作原理简述：
- ZGC 能够做到极致低延迟的秘诀在于：它把 G1 中最耗时的“对象搬家（复制/Evacuation）”阶段，也做成了【并发】的！ 也就是说，垃圾回收器在内存里挪动对象位置的时候，你的业务线程还在同时运行。
- 它依赖两大底层核心技术：
	- 染色指针 (Colored Pointers)：ZGC 直接把对象的引用指针（64位）高位中的几位拿出来，用来存在对象的 GC 状态（比如对象是否被移动过）。
	- 读屏障 (Load Barrier)：当业务代码去读取一个对象引用时，会触发一个轻量级的“读屏障”。如果染色指针显示“这个对象正在被 GC 线程挪动”，读屏障会自适应地帮业务线程“纠正”引用，直接读取新地址，或者顺手帮内核把这个对象复制过去。

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
