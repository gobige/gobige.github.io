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
