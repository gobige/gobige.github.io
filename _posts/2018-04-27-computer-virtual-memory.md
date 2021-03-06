---
layout: post
title: '深入理解计算机系统-虚拟内存'
subtitle: '读书笔记系列深入理解计算机系统之虚拟内存'
date: 2018-04-27
categories: 计算机系统
author: yates
cover: ''
tags: 计算机系统
---

## 前言
一个系统中的**进程**和其他进程**共享cpu和主存资源**，这样的内存很容易被破坏。为了更加有效的管理内存少出错，现代系统使用虚拟内存概念。

**装载的需求**

- 可执行程序加载后占用的内存空间应该是连续的
- 需要同时加载多个程序，但不能让程序自己规定在内存中加载位置

为了解决两个要求，我们把指令用到内存地址叫做虚拟内存地址；实际内存硬件地址，叫做物理内存地址

**分段**
找出一段**连续物理内存**和虚拟内存地址进行映射的方法，叫做分段

**内存交换**
分段解决了程序本身不需关心具体物理内存地址问题，但是会出现**内存碎片**问题。通过内存交换（通过与磁盘swap分区，交换，整理物理内存地址）

**分页**
分段，虚拟内存，内存交换看似美好，解决装载多个程序问题，但是硬盘IO速度时远小于内存的，在大段数据拷贝会造成性能瓶颈；
分页把整个物理内存空间切成一段段固定尺寸大小

```java 
// 查看页大小
$ getconf PAGE_SIZE
```
分页的方式使得在加载程序的时候，不需要一次性把程序都加载到物理内存

**虚拟内存提供的作用**
- 它将主存看成一个存储在磁盘上的地址空间的高速缓存，在主存中只保存活动区域
- 它为每个进程提供了一致的地址空间，从而简化了内存管理
- 保护了每个进程的地址空间不被其他进程破坏

**虚拟内存与物理地址关系**
计算机系统主存被组织成一个由M个连续的字节大小组成的数组。**每个字节有唯一的物理地址**，cpu访问内存就是使用物理地址，称为**物理寻址**.早期PC使用这种方式，现代处理器使用的是**虚拟寻址**，即CPU通过生成一个虚拟地址，这个虚拟地址在送达内存之前先**转换**为适当的物理地址，这个过程叫做**地址翻译**

**地址空间**
一个非负整数地址的有序集合
- 线性地址空间 地址空间中整数是连续的
- 虚拟地址空间 从地址空间中生成虚拟地址
- 物理地址空间 对应于系统中物理内存空间

- 虚拟页 VM系统通过将虚拟内存分割为虚拟页的大小固定的块。
    - 未分配的 VM系统还未分配的页。未分配的块没有任何数据与之关联，因此不占用任何磁盘空间。
    - 缓存的：当前已缓存在物理内存中的已分配页
    - 未缓存的：未缓存在物理内存汇总的已分配页
- 物理页 物理内存被划分为物理页

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/24.png)

**页表**
同任何缓存一样，虚拟内存必须有某种方法判定一个虚拟页**是否缓存**在DRAM中某个地方。以及缓存时虚拟页存放的**物理页**中和不缓存时虚拟页存放物理地址，缓存替换牺牲页，页表将虚拟页映射到物理页，通过**页号和偏移量**映射。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/25.png)

**页命中**
如果在通过PTE读取在虚拟内存时，直接通过PTE读取物理内存读取，如上图VP2，称为页命中

**缺页**
如果在通过PTE读取在虚拟内存时，发现数据并未缓存，则发生缺页，如上图VP3，该程序选择一个牺牲页，并且覆盖

**分配页面**
分配一个新的虚拟内存页时，如下图VP5在磁盘上创建空间并更新PTE5

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/26.png)

整个运行过程中程序引用的不同页面总数可能会超过物理内存总的大小，局部性原则保证了在任意时刻，程序将区域在一个较小的活动页面集合上工作，这个集合叫做工作集。在初始开销，工作集页面会调度到内存中，接下来对工作集引用将导致命中。

操作系统为每个进程提供了一个独立的页表，因而也就是独立的虚拟地址空间。作为内存管理的工具，VM简化了链接和加载，代码和数据共享，以及应用程序的内存分配。并且保证了内存的安全，不允许一个进程修改它的制度代码，不允许它读或修改任何内核中代码和数据结构，不允许它读或者写其他进程的私有内存，不允许它修改任何与其他进程共享的虚拟页面。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/27.png)

每个进程都有属于自己**独立的虚拟内存和内存地址空间**。每个进程都有一个页表。（若**一个页大小4byte**，那么20位高位，12位低位，那么一个进程的**页表的大小就为4byte * 2的20次方=4M**）这样页表所需占用空间就需很大。那么多级页表就出现了。

**多级页表**
在一个实际进程里，虚拟内存占用地址通常是两段连续空间，内存地址从顶往下，不断分配占用栈空间，从底部往上，不断分配占用堆空间。大部分所占用内存空间时有限的，只需存那些用到页之间映射关系就好了

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/28.png) 

多级页表就像一个多叉树结构，虚拟内存的地址连续性，树的第一层很多都是空的，所以不需对应的2级，3级，...页表

如果前20高位分为4级，每一级5byte，第一级有2的5次方（32）条目。每个条目4byte，一共需要128byte，而一个1级索引表，对应32个4KiB的也就是16KB的大小。一个填满的2级索引表，对应的就是32个1级索引表，也就是512KB的大小。

我们可以一起来测算一下，一个进程如果占用了1MB的内存空间，分成了2个512KB的连续空间。那么，它一共需要2个独立的、填满的2级索引表，也就意味着64个1级索引表，2个独立的3级索引表，1个4级索引表。一共需要69个索引表，每个128字节，大概就是9KB的空间。比起4MB来说，只有差不多1/500。

多级页表节约了存储空间，但是带来了时间的开销

**地址翻译**

程序所需指令，顺序放在虚拟内存，通常是一个虚拟页，通过在CPU里放入一个缓存芯片**TLB**,称为**地址变换高速缓冲**，多次内存访问就只需要一次转换。指令和数据分布称为**ITLB**,**DTLB**
地址翻译必须和系统中所有硬件缓存的操作集成，大多数页表条目位于L1高速缓存，但是一个称为TLB的页表条目的片上高速缓存，通常会消除访问在L1上的页表条目开销。

**内存映射**
Linux通过虚拟内存区域与一个磁盘上对象关联起来，对象类型为：
- Linux文件系统中普通文件：例如可执行文件，被分成页大小的片，通过按需进行页面调度。
- 匿名文件：一个区域可以映射一个匿名文件，匿名文件由内核创建，包含全是二进制零。

**进程间共享对象**
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/29.png) 
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/computer-system-Perspective/30.png) 

**动态内存分配**

动态内存分配器维护着一个进程的虚拟内存区域，称为堆。分配器将堆是为一组不同大小的块的集合，每个块就是一个连续的虚拟内存片，要么已分配的，要么空闲的。已分配的显示保留为程序使用，空闲块用来被显式分配（C中malloc），已分配块保持已分配状态，直到被显式（C中free）或隐式释放（java的垃圾回收）。

**碎片**
堆利用率低的主要原因是**碎片**，虽然有未使用的内存但不能用来满足分配请求时，就会发生这种现象。

- 内部碎片，在一个已分配块比有效载荷大的时候发生。
- 外部碎片，当空闲内存合计起来足够满足一个分配请求，但没有一个单独的空闲块足够大可以来处理这个请求时

**垃圾收集**
垃圾收集器将内存视为一张有向可达图，由一个根节点和一组堆节点组成。当不存在一条从任意节点触发到达任意一个堆节点P的有向路径时，我们说P不可达，p会得到释放


## 安全性与内存保护

**可执行空间保护**
无论是指令还是数据，CPU看了都是二进制数据，如果数据部分隐藏了 指令的代码，那么解码后就变成了一条合理指令。对于**进程内存空间执行权限进行控制**只对指令区域代码进行执行，数据区域则拒绝。

**地址空间布局随机化**
进程的内存布局空间时固定的，所以第三方容易知道指令在哪里。对地址空间的区域（栈，堆，数据）进行随机化分配。即使漏洞修改也只是程序crash掉。