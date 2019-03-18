---
layout: post
title: 'jvm性能监控和故障处理工具'
subtitle: 'jvm性能监控和故障处理工具'
date: 2018-03-09
categories: jvm
author: yates
cover: ''
tags: jvm
---

### 前言
jdk团队研发了很多用于jvm性能监控的工具，下面我们来看看这些工具的用途和使用


这些工具都位于jdk/lib/tools.jar类库里，可通过调用该类库的接口实现生产环境的监控。
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/10.png)
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/11.png)

**jps：jvm进程状况工具**
- 列出正在运行的虚拟机进程，并显示虚拟机执行主类

jps指令常见选项
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/12.png)

**jstat：虚拟机统计信息监视工具**
- 用于监视jvm各种运行专题信息，显示类装载，内存，垃圾回收，jit编译等运行数据

jstat指令常见选项
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/13.png)


```java
jstat -gcutil 26797 2000 100  // 每2秒查询26797进程GC情况 共100次 
```
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/38.png)

**各项意义**
S0:存活区1
S1:存活区2
E:年轻代
O:老年代
M:元空间 (对应以前的P 永久代,方法区)
YGC:minorGC次数
YGCT:minorGC总耗时
FGC:fullGC次数
FGCT:fullGC总耗时
GCT:GC总耗时


**jinfo：java配置信息工具**
- 实时查看和调整jvm各项参数

**jmap：java内存映像工具**
- 用于生成堆转储快照，查询finalize执行队列，堆和永久代详细信息，jvm发生oom时的dump文件
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/14.png)

**jhat：jvm堆转储快照分析工具**
- 通过内置的HTTP/HTML服务器，配和jmap生成的dump文件，生成分析文件；这个过程是一个大量消耗硬件资源，建议不要在生产环境中使用

**jstack：java堆栈跟踪工具**
- 用于生成jvm当前时刻线程快照，显示每一条现场正在执行方法堆栈集合，从而可用定位线程可能出现诸如死锁，死循环，请求外部资源长时间等待等问题
![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/15.png)

除了上述命令行工具，jdk还提供了两个强大的外部可视化工具JConsole，VisualVM

**JConsole：java监视和管理控制台**

![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2018-03-19-jvm/16.png)

**VisualVM：多合一故障处理工具**
VisualVM提供很多插件的扩展，可以做到显示进程和进程配置，监视应用CPU，GC，堆，方法区，提供dump以及堆转储文件分析；方法级程序性能分析，找出调用最多，运行时间最长方法；收集程序运行时配置，线程dump，内存dump


禁止System.gc()
```java
-XX:+DisableExplicitGC
```

### JVM常见配置参数

**栈设置** 
每个线程的栈大小
```java
-Xss128k
```

**堆设置** 
初始堆大小
```java
-Xms
```
最大堆大小
```java
-Xmx
```
设置年轻代大小
```java
-Xmn
-XX:NewSize=1024 
-XX:MaxNewSize 
```
设置年轻代和年老代的比值.如:为3,表示年轻代与年老代比值为1:3,年轻代占整个年轻代年老代和的1/4
```java
-XX:NewRatio=n
```
年轻代中Eden区与两个Survivor区的比值.注意Survivor区有两个.如:3,表示Eden:Survivor=3:2,一个Survivor区占整个年轻代的1/5
```java
-XX:SurvivorRatio=n
```
设置持久代大小
```java
-XX:PermSize
-XX:MaxPermSize
```
一个对象最大在Survivor区存活age
```java
-XX:MaxTenuringThreshold=
```

**收集器设置**
设置串行收集器
```java
-XX:+UseSerialGC
```
设置并行收集器
```java
-XX:+UseParallelGC
```
设置并行年老代收集器
```java
-XX:+UseParallelGC
```
设置并发收集器
```java
-XX:+UseParallelGC
```
CMS收集，设置年老代为并发收集
```java
-XX:+UseConcMarkSweepGC
```
**垃圾回收统计信息**
```java
-XX:+PrintGC 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps 
```

**内存溢出时生成堆dump文件**
```java
-XX:+HeapDumpOnOutOfMemoryError
```java

jit编译的代码都是放在codecache
```java
-XX:+UseCodeCacheFlushing 最早被编译的一半方法将会被放到一个old列表中等待回收
在一定时间间隔内，如果方法没有被调用，这个方法就会被从codeCache充清除
 -XX:ReservedCodeCacheSize  设置codecache大小
```
