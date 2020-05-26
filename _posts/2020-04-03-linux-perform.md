---
layout: post
title: 'Linux性能优化'
subtitle: 'Linux性能优化'
date: 2020-04-03
categories: Linux
author: yates
cover: 'www.baidu.com'
tags: Linux
---

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/2.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/3.png)

## 性能分析常用步骤

- 选择指标评估应用程序和系统性能
- 为应用程序和系统设置性能目标
- 进行性能基准测试
- 性能分析定位瓶颈
- 优化系统和应用程序
- 性能监控和告警

# 平均负载


单位时间内，系统处于**可运行状态**和**不可中断状态**的平均进程数，**平均活跃进程数**

- 可运行状态进程：正在使用或等待CPU的进程
- 不可中断状态进程：正处于内核态关键流程中的进程，且不可中断，例：等待硬件设备I/O响应

当平均负载高于 CPU 数量70%的时候，你就应该分析排查负载高的问题了


**相关指令**

- uptime：当前时间；系统运行时间；正登陆用户；CPU平均负载
- ps：查看进程运行状态 R-可运行状态进程；D-不可中断状态进程
- grep 'model name' /proc/cpuinfo | wc -l：查看cpu核心数

**相关工具**

- stress：（stress --cpu 1 --timeout 600）linux系统压力测试工具模拟异常进程
- sysstat：常用的 Linux 性能工具
	- pidstat：（pidstat -u 5 1 间隔 5S输出 cpu1的数据）常用进程性能分析工具，实时查看进程CPU,内存，I/O，上下文切换指标
	- mpstat：（mpstat -P ALL 5 监控所哟CPU，每5S输出一组数据）多核cpu性能分析工具，用于查看每个CPU性能指标

```java
stress --cpu 1 --timeout 600 // 构造 一个 CPU 密集型进程

stress -i 1 --timeout 600 // 构造一个 I/O 密集型进程

stress -c 8 --timeout 600  构造一个 大量进程的场景
```

## CPU上下文切换

前一个任务的**CPU上下文（CPU寄存器和程序计数器）**，保存到**系统内核**中。加载新任务上下文，跳到程序计数器新位置。

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/4.png)

内核空间（Ring 0）具有最高权限，可直接访问所有资源

用户空间（Ring 3）只能访问受限资源，不能直接访问内存等硬件设备

进程既可以在用户，也可以在内核空间运行，分别为**用户态，内核态**。需要通过**系统调用**来完成

一次**系统调用**的过程，一直在**同一个进程**中运行，其实发生了两次**CPU上下文**切换（用户态和内核态的切换），通常称为特权模式切换，而不是上下文切换。


**进程上下文切换**

每次进程上下文切换都需要**几十纳秒**到**数微秒**CPU时间，大量时间耗费寄存器，内核栈以及虚拟内存**资源的保存和恢复**，还会导致**TLB的刷新**，**高速缓存的刷新**。

进程上下文切换原因：

- 时间片耗尽
- 资源不足
- sleep函数
- 高优先级进程插入

**线程上下文切换**

线程是**调度**的基本单位，进程是**资源**拥有的基本单位。

切换两种情况：线程所属不同进程。线程属于同一进程

**中断上下文切换**

为了快速响应硬件事件，**中断能处理会打断进程正常调度和执行**

**相关指令**

vmstat指令：分析系统内存使用情况，CPU上下文切换和中断次数
pstree | grep stress : 用树形形式显示所哟进程间关系

[点这里-> linux指令大全](http://muyibeyond.cn/2017/11/12/linux-file.html)

查看具体进程上下文切换情况：pidstat -w 5 // -t 显示线程上下文次数

```java
UID       PID   cswch/s nvcswch/s  Command
0         1      0.20      0.00  systemd
0         8      5.40      0.00  rcu_sched

cswch:每秒自愿上下文切换 常指 进程无法获取所需资源
nvcswch:每秒非自愿上下文切换 常指 时间片等原因
```

**测试工具**

```java
$ sysbench --threads=10 --max-time=300 threads run // 10个进程运行5分钟基准测试，模拟多线程切换问题
```

```java
watch -d cat /proc/interrupts // /proc 实际上是 Linux 的一个虚拟文件系统，用于内核空间与用户空间之间的通信。/proc/interrupts 就是这种通信机制的一部分，提供了一个只读的中断使用情况

该显示信息中会有：重调度中断（RES）---表示唤醒空闲状态CPU来调度新任务运行，
多处理器系统（SMP）中，调度器用来分散任务到不同 CPU 的机制，通常也被称为处理器间中断（Inter-Processor Interrupts，IPI）
```

- 自愿上下文切换变多，说明进程都在等待资源，有可能发生I/O等问题
- 非自愿上下文切换变多，说明进程被强制调度，都在争抢CPU，说明CPU的确成了瓶颈
- 中断次数变多，CPU被中断处理占用，须通过 /proc/interrupts 查看具体原因


## CPU使用率

Linux通过事先定义**节拍率（内核表示为HZ）**，触发时间中断，并使用全局变量Jiffies记录开机以来节拍数 

```java
grep 'CONFIG_HZ=' /boot/config-$(uname -r) // 查看CPU节拍率

cat /proc/stat | grep ^cpu // 向用户空间提供 CPU相关信息

显示信息 依次 user：用户态 CPU 时间；nice：低优先级用户态 CPU 时间；system：内核态 CPU 时间；
idle：空闲时间；iowait：等待 I/O 的 CPU 时间；irq：处理硬中断的 CPU 时间；softirq：软中断的 CPU 时间；
steal：当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间；guest：运行虚拟机的 CPU 时间；
guest_nice：低优先级运行虚拟机的时间
```

平均CPU使用率 = 1 - (空闲时间new - 空闲时间old)/(总CPU时间new - 总CPU时间old)

Linux 也给每个进程提供了运行情况的统计信息，也就是 /proc/[pid]/stat

**相关指令**

top：显示系统总体CPU和内存使用情况，各个进程资源使用情况 // CPU使用率包含了用户态和内核态，具体细分要用pidstat
ps：显示每个进程资源使用情况


[点这里-> linux指令大全](http://muyibeyond.cn/2017/11/12/linux-file.html)

perf：可以指定进程或时间进行采样，调用栈形式，输出整个调用链汇总信息

perf top：实时显示占用CPU时钟最多函数或指令
perf record：保存数据
perf report：解析数据

```java
$ perf top

// 采样数（Samples）、事件类型（event）和事件总数量（Event count）
Samples: 833  of event 'cpu-clock', Event count (approx.): 97742399 
Overhead  Shared Object       Symbol
   7.28%  perf                [.] 0x00000000001f78a4
   4.72%  [kernel]            [k] vsnprintf
   4.32%  [kernel]            [k] module_get_kallsym
   3.65%  [kernel]            [k] _raw_spin_unlock_irqrestore
...

Shared ，是该函数或指令所在的动态共享对象（Dynamic Shared Object），如内核、进程名、动态链接库名、内核模块名等
Object ，是动态共享对象的类型。比如 [.] 表示用户空间的可执行程序、或者动态链接库，而 [k] 则表示内核空间
Symbol 是符号名，也就是函数名
```

**相关工具**

execsnoop 就是一个专为**短时进程**设计的工具。它通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括**进程 PID、父进程 PID、命令行参数以及执行的结果**

碰到常规问题无法解释的 CPU 使用率情况时，首先要想到有可能是短时应用导致的问题

- 用户 CPU 和 Nice CPU 高，说明用户态进程占用了较多的 CPU，所以应该着重排查进程的性能问题。
- 系统 CPU 高，说明内核态占用了较多的 CPU，所以应该着重排查内核线程或者系统调用的性能问题。
- I/O 等待 CPU 高，说明等待 I/O 的时间比较长，所以应该着重排查系统存储是不是出现了 I/O 问题。
- 软中断和硬中断高，说明软中断或硬中断的处理程序占用了较多的 CPU，所以应该着重排查内核中的中断服务程序。

## 线程状态

- R：表示进程在CPU的就绪队列中，**运行或正在等待运行**
- D：**不可中断状态睡眠**，一般表示进程正跟硬件交互  // 若是系统或硬件发生故障，进程可能会在不可中断状态保持很久，导致系统出现大量不可中断进程
- Z：僵尸进程（进程结束了，但是父进程没有回收它资源，描述符，PID）// 父进程创建子进程后，通过wait或waitpid等待子进程结束；子进程结束时，会向它父进程发送SIGCHLD信号；父进程可以注册处理函数，异步回收资源；父进程没来得及处理或子进程执行太快都会导致僵尸进程的出现
- S：**可中断状态睡眠**，进程**等待某个事件**被系统挂起。
- I：空闲状态，不可中断睡眠内核线程上。除了D状态的进程状态
- T/t：进程处于暂停或跟踪状态
- X：进程已消亡


**相关工具**

dstat：同时观察系统CPU，磁盘I/O，网络以及内存使用情况


**进程组和会话**

- 进程组表示**一组相互关联的进程**，每个子进程都是父进程所在组成员
- 会话指共享同一个控制终端的**一个或多个进程组**

iowait 高不一定代表I/O 有性能瓶颈。当系统中**只有 I/O 类型的进程在运行**时，**iowait也会很高**，但实际上，磁盘的读写远没有达到性能瓶颈的程度。
 
## 中断

- **中断**其实是一种**异步**的事件处理机制，可以**提高系统的并发**处理能力；为了减少对正常进程运行影响，中断处理需尽可能快运行
- 中断处理程序在响应中断时，会临时关闭中断

**软中断**

为了解决中断处理程序执行过长和中断丢失，中断处理过程分成两个阶段

- 上半部分**快速处理中断**，**硬中断**
- 下半部分用来延迟上半部分未完成工作，通常以**内核线程**方式运行，**软中断**。一些内核自定义事件也属于软中断

```java
/proc/softirqs 提供了软中断的运行情况；
/proc/interrupts 提供了硬中断的运行情况。
```

Linux 中，**每个CPU**都对应一个**软中断内核线程**，名字是ksoftirqd/CPU编号.**软中断**事件的**频率过高**时，内核线程也会因为CPU使用率过高而导致**软中断处理不及时**，进而引发**网络收发延迟、调度缓慢**等性能问题。


**相关工具**

- sar 是一个系统活动报告工具，既可以实时查看系统的当前活动，又可以配置保存和报告历史统计数据(需安装sysstat软件包)
- hping3 是一个可以构造 TCP/IP 协议数据包的工具，可以对系统进行安全审计、防火墙测试等
- tcpdump 是一个常用的网络抓包工具，常用来分析各种网络问题。


## 性能优化方法论

**优化前思考**

1. 既然要做性能优化，那要判断优化是否**有效**？能**提升多少**性能？
2. 多个性能问题，先**优化哪一个**
3. 提升性能方法很多，**使用哪一个呢**，性能优化成本

**怎么评估优化效果**

1. 确定性能量化指标
2. 测试优化前性能指标
3. 测试优化后性能指标

应用程序的维度（吞吐量和请求延迟），系统资源的维度（CPU使用率）

**注意**：
1避免性能测试工具干扰程序性能 
2避免外部环境变化  
3要忍住“把 CPU 性能优化到极致”的冲动


## Linux内存管理

Linux内核给每个进程提供一个连续的独立的虚拟地址空间。虚拟地址分为**内核空间**和**用户空间**。
**实际使用的虚拟内存**才分配物理内存，分配后，通过**内存映射**来管理。内存映射使用页表来记录。

**页表**

页表实际上存储在CPU内存管理单元MMU，当进程访问虚拟地址在页表中找不到，系统产生一个**缺页异常**，
进入内核空间分配物理内存，更新页表，再返回用户空间，恢复进程运行。

TLB是MMU中页表高速缓存，减少进程上下文切换，减少TLB刷新次数，提高TLB缓存使用率，提高CPU内存访问性能

MMU通过页而不是字节管理内存，页的大小4KB，为了解决页表项过多问题，Linux提供两种机制，**多级页表和大页**

多级页表：把内存分成**区块**来管理，将原来映射关系改为**区块索引和区块偏移**


**虚拟内存空间分布**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/7.png)

- 只读段：包括代码和常量
- 数据段：包括全局变量
- 堆：包括动态分配内存，低地址向上增长。
- 文件映射段，动态库，共享内存等，高地址向下增长。
- 栈，包括局部变量和函数调用上下文。


使用C标准malloc函数或使用mmap函数进行动态分配，小块内存（小于128K），C 标准库使用 brk() 来分配，大块内存（大于 128K），则直接使用内存映射 mmap() 来分配

malloc() 申请虚拟内存后，系统并不会立即为其分配物理内存，而是在首次访问时，才通过缺页异常陷入内核中分配内存


**buff/cache**

我们使用free指令时在输出信息中有 buff/cache 列：

- buffers:内核缓冲区用到的内存，对应/proc/meminfo中buffers值，文件中表示--对原始磁盘块临时存储，用来**缓存磁盘数据**。
- cache：内核页缓冲和Slab用到的内存，对应/proc/meminfo中Cached与SReclaimable之和。Cached文件中表示--从磁盘读取文件页缓存，**缓存从文件读取数据**
- SReclaimable是Slab一部分。Slab包括两部分，可回收部分用SReclaimable记录；不可回收部分用SUnreclaim记录


Buffer既可以用作“将要写入磁盘数据的缓存”，也可以用作“从磁盘读取数据的缓存”。
Cache既可以用作“从文件读取数据的页缓存”，也可以用作“写文件的页缓存”。

Buffers 和 Cache 都是操作系统来管理的，应用程序并不能直接控制这些缓存的内容和生命周期

同一个指标具体含义，因为内核版本，性能工具版本不同有挺大差别。

**相关工具**
安装 bcc-tools工具

- cachestat：提供整个操作系统缓存读写命中情况

```java
## 缓存命中的次数  缓存未命中的次数  新增到缓存中的脏页数  命中率 Buffers大小  Cache大小
HITS   MISSES  DIRTIES HITRATIO   BUFFERS_MB  CACHED_MB
   0        0        0    0.00%          207       9810
   3        0        4  100.00%          207       9810
   0        0        0    0.00%          207       9810
   0        0        4    0.00%          207       9810
```

- cachetop：提供每个进程缓存命中情况

```java
 
10:57:40 Buffers MB: 207 / Cached MB: 9811 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
    2013 root     wrapper                 2        0        0     100.0%       0.0%
    2425 root     Wrapper-Connect         2        0        0     100.0%       0.0%
   25326 root     java                    2        1        1	   33.3%       0.0%
```

- pcstat：基于Go语言开发，查看文件在内存中的缓存大小及缓存比例(需安装Go语言)
- dd：磁盘和文件拷贝工具，经常拿来测试磁盘或文件读写性能。
- memleak：可以跟踪系统或指定进程的内存分配，释放请求，然后定期输出一个未释放内存和相应调用栈汇总情况




根据Linux内核eBPF机制，跟踪内核中管理缓存，并输出缓存使用和命中。

内存泄漏会导致两种可能结果：

- 内存回收：系统释放掉可以回收内存，比如：缓冲区和缓存。被叫做文件页。
	- 大部分文件页，直接回收；
	- 那些被修改过，暂时没写入磁盘的数据（脏页），得先写入磁盘
		- 通过系统调用fsync，把脏页同步到磁盘
		- 交给系统，由内核线程pdflush负责脏页刷新
- OOM杀死进程

**Swap原理**


Swap把不常访问内存先写到磁盘中，然后释放内存给其他进程使用。

把一块磁盘或一个本地文件当成内存来使用，包括换入和换出两个过程

- 换出：把进程暂时不用内存存储到磁盘，并释放这些数据占用内存
- 换入：进程再次访问时，把它们从磁盘读到内存

**什么时候回收**

- 直接内存回收：比如，有新的大块内存分配请求，但剩余内存不足，系统回收内存中缓存
- kswapd0：
	- 页最小阈值：剩余内存小于该值时,进程内存耗尽，只有内核能分配内存
	- 页低阈值：剩余内存小于该值时，kswapd0执行内存回收，直到剩余内存大于页高阈值
	- 页高阈值：剩余内存小于该值时，内存有压力，但能满足新内存请求；大于时没有内存压力

页低阈值通过内核选项 /proc/sys/vm/min_free_kbytes 的 min_free_kbytes参数 来间接设置，其他两个阈值，根据以下计算生成的

```java
pages_low = pages_min*5/4
pages_high = pages_min*3/2
```

三个内存阈值通过proc 文件系统中的接口 /proc/zoneinfo 来查看

/proc/zoneinfo
 
**NUMA与Swap**

NUMA 架构下，多个处理器被划分到不同 Node 上，且每个 Node 都拥有自己的本地内存空间

一个 Node 内部的内存空间，实际上又可以进一步分为不同的内存域（Zone），比如直接内存访问区（DMA）、普通内存区（NORMAL）、伪内存区（MOVABLE）等

```java
numactl --hardware  // 通过 numactl 命令，来查看处理器在 Node 的分布情况,以及每个 Node 的内存使用情况
```

通过 /proc/sys/vm/zone_reclaim_mode 来调整回收模式

- 0，表示既可以从其他 Node 寻找空闲内存，也可以从本地回收内存。
- 1、2、4 都表示只回收本地内存，2 表示可以回写脏数据回收内存，4 表示可以用 Swap 方式回收内存。

通过 /proc/sys/vm/swappiness 选项，用来调整使用Swap的积极程度，范围是0-100，数值越大，越积极使用Swap。
但是即使你把它设置成0，当剩余内存+文件页小于页高阈值时，还是会发生Swap


**通常，降低Swap的使用，可以提高系统的整体性能**

- 禁止Swap，现在服务器的内存足够大，所以除非有必要，禁用Swap就可以了
- 实在需要用到Swap，可以尝试降低swappiness的值，减少内存回收时Swap的使用倾向

## Linux文件系统

Linux中一切皆文件。不仅普通的文件和目录，就连块设备、套接字、管道等，也都要通过统一的文件系统来管理

**目录项和索引节点**

每个文件都分配两个数据结构：

- 索引节点：inode编号、文件大小、访问权限、修改日期、数据的位置等，索引节点和文件一一对应
- 目录项：由内核维护一个**内存数据结构**，记录文件的名字、索引节点指针以及与其他目录项的关联关系。多个关联的目录项，就构成了文件系统的目录结构

目录项和索引节点的关系是多对一

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/8.png)

- 目录项本身是一个内存缓存，索引节点存储在磁盘中数据。文件内存会缓存到页缓存Cache中
- 磁盘被系统格式化三个存储区域
	- 超级块，存储整个文件系统状态
	- 索引节点区，存储索引节点
	- 数据块区，存储文件数据
	
**虚拟文件系统**

为了支持各种不同的文件系统，Linux内核在用户进程和文件系统的中间，引入虚拟文件系统VFS

文件系统可以分为三类：

- 基于磁盘的文件系统：把数据直接存储在计算机本地挂载的磁盘
- 基于内存的文件系统：，不需要任何磁盘分配存储空间，但会占用内存，例：/proc ，/sys文件系统
- 网络文件系统：访问其他计算机数据的文件系统

文件系统，要先挂载到 VFS 目录树中的某个子目录（称为挂载点），然后才能访问其中的文件

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/9.png)

**文件读写方式分类**

是否利用标准库缓存

- 缓冲I/O：利用标准库缓存来加速文件的访问，而标准库内部再通过系统调度访问文件
- 非缓冲I/O：通过系统调用来访问文件

是否利用操作系统的页缓存

- 直接I/O：跳过操作系统的页缓存，直接跟文件系统交互来访问文件
- 文件读写时，先要经过系统的页缓存，然后再由内核或额外的系统调用

应用程序是否阻塞自身运行

- 阻塞I/O：应用程序执行I/O操作后，如果没有获得响应，阻塞当前线程，不能执行其他任务
- 非阻塞I/O：应用程序执行I/O操作后，当前的线程可以继续执行其他的任务，随后再通过轮询或者事件通知的形式，获取调用的结果

是否等待响应结果

- 同步I/O：执行I/O操作后，一直等到整个I/O完成，才能获得I/O响应
- 异步I/O：执行I/O操作后，继续执行其他任务，I/O完成后，事件通知的方式，通知应用程序

查看磁盘容量

```java
df -i /dev/sda1 
```

当你发现索引节点空间不足，但磁盘空间充足时，很可能就是过多小文件导致（删除小文件或移到索引节点充足磁盘）

查询目录项和各种文件系统索引节点的缓存情况

```java
cat /proc/slabinfo | grep -E '^#|dentry|inode' 

slabtop 
```

在读写普通文件时，I/O请求先经过文件系统，再与磁盘交互。而读写块设备时，跳过文件系统，直接与磁盘交互，也所谓“裸I/O”

**通用块层**

处在文件系统和磁盘驱动中间的一个块设备抽象层。

- 向上，为文件系统和应用程序，提供访问块设备的标准接口；向下，把各种异构的磁盘设备抽象为统一的块设备，并提供统一框架来管理这些设备的驱动程序
- 文件系统和应用程序发来的 I/O 请求排队，并通过重新排序、请求合并等方式，提高磁盘读写的效率


**Linux 内核支持四种I/O调度算法**

- NONE：对文件系统和应用程序的I/O其实不做任何处理
- NOOP：一个先入先出的队列，做一些最基本的请求合并，常用于 SSD 磁盘
- CFQ：为每个进程维护了一个 I/O 调度队列，并按照时间片来均匀分布每个进程的 I/O 请求
- DeadLine分别为读、写请求创建了不同的 I/O 队列，可以提高机械磁盘的吞吐量，并确保达到最终期限（deadline）的请求被优先处理。多用于数据库等I/O压力重场景


## 磁盘性能指标 

- 使用率：磁盘处理I/O的时间百分比
- 饱和度：磁盘处理 I/O 的繁忙程度
- IOPS：每秒的 I/O 请求数
- 吞吐量：每秒的 I/O 请求大小
- 响应时间I/O 请求从发出到收到响应的间隔时间


不同应用场景，要考虑不同的I/O指标，结合读写比例，I/O类型（read，write），I/O大小，综合分析

用性能工具得到的这些指标，可以作为后续分析应用程序性能的依据。一旦发生性能问题，你就可以把它们作为磁盘性能的极限值，进而评估磁盘 I/O 的使用情况


**相关工具**

iostat：提供了每个磁盘的使用率、IOPS、吞吐量等各种常见的性能指标，指标来自 /proc/diskstats。

[iostat指令](http://muyibeyond.cn/2017/11/12/linux-file.html)

%util：磁盘I/O使用率；
r/s+ w/s：IOPS；
rkB/s+wkB/s：吞吐量；
r_await+w_await：响应时间。


```java
pidstat -d 1 // 观察进程的I/O情况
iotop // 观察进程的I/
strace -p 18940 // 分析系统调用  
lsof -p 18940  // 查看进程打开文件列表 包括了目录、块设备、动态库、网络套接字等
 ./filetop -C  // 跟踪内核中文件的读写情况，并输出线程ID（TID）、读写大小、读写类型以及文件名称
opensnoop  // 动态跟踪内核中的 open 系统调用
```

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/11.png)
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/12.png)


## 磁盘I/O性能优化

**I/O基准测试**
fio是常用的文件系统和磁盘I/O性能基准测试工具

```java

// direct 是否跳过系统缓存  1 表示跳过系统缓存
// iodepth  使用异步I/O  发出的 I/O 请求上限
// rw，  I/O 模式  顺序/随机/读/写
// ioengine I/O 引擎,支持同步（sync）、异步（libaio）、内存映射（mmap）、网络（net）等各种 I/O 引擎
// bs   表示I/O 的大小
// filename  文件路径   磁盘路径(用磁盘路径测试写，会破坏这个磁盘中的文件系统)/文件路径

# 随机读
fio -name=randread -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 随机写
fio -name=randwrite -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序读
fio -name=read -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序写
fio -name=write -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb 
```

测试结果例：
```java
read: (g=0): rw=read, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [R(1)][100.0%][r=16.7MiB/s,w=0KiB/s][r=4280,w=0 IOPS][eta 00m:00s]
read: (groupid=0, jobs=1): err= 0: pid=17966: Sun Dec 30 08:31:48 2018
   read: IOPS=4257, BW=16.6MiB/s (17.4MB/s)(1024MiB/61568msec)
    slat (usec): min=2, max=2566, avg= 4.29, stdev=21.76 // 指从 I/O 提交到实际执行 I/O 的时长（Submission latency）
    clat (usec): min=228, max=407360, avg=15024.30, stdev=20524.39 // 指从 I/O 提交到 I/O 完成的时长（Completion latency）
     lat (usec): min=243, max=407363, avg=15029.12, stdev=20524.26 // 指的是从fio 创建 I/O 到 I/O 完成的总时长
    clat percentiles (usec):
     |  1.00th=[   498],  5.00th=[  1020], 10.00th=[  1319], 20.00th=[  1713],
     | 30.00th=[  1991], 40.00th=[  2212], 50.00th=[  2540], 60.00th=[  2933],
     | 70.00th=[  5407], 80.00th=[ 44303], 90.00th=[ 45351], 95.00th=[ 45876],
     | 99.00th=[ 46924], 99.50th=[ 46924], 99.90th=[ 48497], 99.95th=[ 49021],
     | 99.99th=[404751]
   bw (  KiB/s): min= 8208, max=18832, per=99.85%, avg=17005.35, stdev=998.94, samples=123 // 代表吞吐量
   iops        : min= 2052, max= 4708, avg=4251.30, stdev=249.74, samples=123
  lat (usec)   : 250=0.01%, 500=1.03%, 750=1.69%, 1000=2.07% // 每秒 I/O 的次数
  lat (msec)   : 2=25.64%, 4=37.58%, 10=2.08%, 20=0.02%, 50=29.86%
  lat (msec)   : 100=0.01%, 500=0.02%
  cpu          : usr=1.02%, sys=2.97%, ctx=33312, majf=0, minf=75
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwt: total=262144,0,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=16.6MiB/s (17.4MB/s), 16.6MiB/s-16.6MiB/s (17.4MB/s-17.4MB/s), io=1024MiB (1074MB), run=61568-61568msec

Disk stats (read/write):
  sdb: ios=261897/0, merge=0/0, ticks=3912108/0, in_queue=3474336, util=90.09% 
```

借助 blktrace，记录磁盘设备的I/O访问情况；然后使用 fio ，重放 blktrace 的记录,实现对应用程序 I/O 模式的基准测试

## I/O 性能优化

**应用程序优化**

- 用追加写代替随机写，减少寻址开销，加快 I/O 写的速度
- 借助缓存 I/O ，充分利用系统缓存，降低实际 I/O 的次数
- 应用程序内部构建自己的缓存，或者用 Redis 这类外部缓存系统
- 频繁读写同一块磁盘空间时，可以用 mmap 代替 read/write，减少内存的拷贝次数
- 同步写的场景中，尽量将写请求合并，而不是让每个请求都同步写入磁盘，即可以用 fsync() 取代 O_SYNC
- 多个应用程序共享相同磁盘时，为了保证 I/O 不被某个应用完全占用，推荐你使用 cgroups 的 I/O 子系统，来限制进程 / 进程组的 IOPS 以及吞吐量

**文件系统优化**

- 根据实际负载场景的不同，选择最适合的文件系统
- 优化文件系统的配置选项，包括文件系统的特性,日志模式、挂载选项
- 优化文件系统的缓存

**磁盘优化**

- 换用性能更好的磁盘，比如用 SSD 替代 HDD
- 使用 RAID ，把多块磁盘组合成一个逻辑磁盘，，提高数据的可靠性，又可以提升数据的访问性能
- 选择最适合的 I/O 调度算法
- 对应用程序的数据进行磁盘级别的隔离。例：为日志、数据库等I/O压力比较重的应用，配置单独的磁盘
- 顺序读比较多的场景中，我们可以增大磁盘的预读数据
- 优化内核块设备 I/O 的选项

## Linux 网络

### **Linux网络栈**

根据TCP/IP模型，数据包按照协议栈，对上层数据进行逐层处理，封装，发送到下一层。网络接口配置最大传输单元（MTU）规定了最大数据包大小

### **Linux网络收发流程**

**接收**

1. 网络帧到达网卡后，网卡通过**DMA**方式，把**网络包放入收包队列(环形缓冲区)**。然后硬中断，告诉中断处理程序已收到网络包
2. **中断处理程序**为网络帧分配**内核数据结构（sk_buff）**，将其拷贝到sk_buff缓冲区，然后通过软中断，通知内核。
3. **内核协议栈**从缓冲区取出网络帧，通过网络栈**逐层处理帧包**
    - 链路层检查报文合法性，去掉帧头，帧尾，交给网络层
    - 网络层取出IP头，判断网络包下一步走向（转发，交由上层），若是后者，去掉IP头，交由传输层
    - 传输层根据端口IP四元组，找到对应socket，并把数据拷贝到Socket接收缓存中

**发送**

跟接收流程相反

1. 调用**Socket API发送网络包**（切换到**内核态**，套接字层把数据包放到**Sokcet发送到缓存区**）
2. **网络协议栈**从缓冲区取出数据包，**逐层封装**，对网络包按照**MTU大小分片**。
3. 网络包送到网络接口层，进行物理地址寻址，找到**下一跳MAC地址**。**添加帧头和帧尾**，放到**发包队列**。然后软中断通知驱动程序
4. 驱动程序通过**DMA**，从发包队列**读出网络帧**，通过**物理网卡发出**

这些缓冲区都处于内核管理的内存中

sk_buff、套接字缓冲、连接跟踪等，都通过slab分配器来管理。你可以直接通过/proc/slabinfo，来查看它们占用的内存大小

### 性能指标

- 带宽：链路的最大传输速率，比特/秒
- 吞吐量：没丢包时的最大数据传输速率
- 网络的使用率：吞吐量/带宽
- 延时：网络请求发出，一直到收到远端响应，需要的时间延迟
- PPS：以网络包为单位的传输速率。通常硬件交换机好于Linux服务器的转发

**查看网络配置**
```java
ifconfig eth0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500 // RUNNING 表示物理网络连通
        inet 10.20.20.170  netmask 255.255.255.0  broadcast 10.20.20.255
        inet6 fe80::f816:3eff:fe4e:8817  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:4e:88:17  txqueuelen 1000  (Ethernet)
        RX packets 34282995  bytes 33190146394 (30.9 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 24099082  bytes 7036560497 (6.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
// RX，TX：

errors--表示发生错误数据包数（校验错误，帧同步）；
dropped--丢弃的数据包数（内存不足）
overruns--超限数据包数（网络I/O速度过快，队列满导致的丢包）
carrier--发生carrirer错误的数据包数（双工模式不匹配、物理电缆出现问题）
collisions--碰撞数据包数
```

**查看套接字，网络栈，网络接口，路由表信息**

```java
# -l 表示只显示监听套接字
# -t 表示只显示 TCP 套接字
# -n 表示显示数字地址和端口(而不是名字)
# -p 表示显示进程信息
$ ss -ltnp | head -n 3
```

- 套接字处于连接状态（Established）
    - Recv-Q 表示套接字缓冲还没有被应用程序取走的字节数（即接收队列长度）
    -  Send-Q 表示还没有被远端主机确认的字节数（即发送队列长度）
- 套接字处于监听状态（Listening）
    - Recv-Q 表示 **syn backlog(半连接队列长度:还未收到客户端ACK)** 的当前值
    - Send-Q 表示**最大**的 syn backlog 值


**查看协议栈统计信息**

```java
 netstat -s
```

**查看网络吞吐和PPS**

```java
sar -n DEV 1
 
Linux 4.15.0-1035-azure (ubuntu) 	01/06/19 	_x86_64_	(2 CPU)

接收和发送的 PPS；接收和发送的吞吐量；接收和发送的压缩数据包数；网络接口的使用率
13:21:40        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
13:21:41         eth0     18.00     20.00      5.79      4.25      0.00      0.00      0.00      0.00
13:21:41      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00


Bandwidth 可以用 ethtool

？？？ ethtool eth0 | grep Speed
```

**连通性和延时**

```java
ping -c3 114.114.114.114
```
Nginx采用工作模型

**采用主进程+多个worker子进程的工作模型**

- 主进程执行 bind() + listen() 后，主进程初始化套接字，创建多个子进程，管理子进程生命周期
- 子进程中，都通过 accept() 或 epoll_wait() ，来处理相同的套接字，处理实际请求 


**accept，epoll_wait引发惊群问题，Nginx通过全局锁来解决惊群问题**

**采用监听到相同端口的多进程模型**

内核保证了只有进程被唤醒，不会出现惊群问题

**C10K**问题 依靠I/O多路复用，请求处理优化可以解决

**C100K C1000K**问题 从应用程序到Linux内核，CPU，内存，网络各个层次深度优化，特别是增加硬件资源，提高硬件性能

**C10M**问题 跳过内核协议栈的冗长路径，把**网络包直接送到应用程序**

- DPDK：用户态网络的标准。用户态进程通过轮询的方式，来处理网络接收
- XDP： Linux 内核提供一种高性能网络数据路径。网络包，在进入**内核协议栈之前，就进行处理**，也可以带来更高的性能

### 网络基准测试

评估网络性能，应该清楚应用程序基于协议栈哪一层？需要测试什么层的网络性能？

**转发性能**

**相关工具**

hping3：测试网络包处理能力
pktgen：根据实际需要构造所需网络包，准确地测试出目标服务器的性能。（可能需编译内核，劝退）

**TCP/UDP 性能**

**相关工具**

**iperf**
```java
# -s表示启动服务端，-i表示汇报间隔，-p表示监听端口
$ iperf3 -s -i 1 -p 10000

# -c表示启动客户端，192.168.0.30为目标服务器的IP
# -b表示目标带宽(单位是bits/s)
# -t表示测试时间
# -P表示并发数，-p表示目标服务器监听端口
$ iperf3 -c 192.168.0.30 -b 1G -t 15 -P 2 -p 10000
```

result 
```java

测试时间、数据传输量以及带宽 

[ ID] Interval           Transfer     Bandwidth
...
[SUM]   0.00-15.04  sec  0.00 Bytes  0.00 bits/sec                  sender
[SUM]   0.00-15.04  sec  1.51 GBytes   860 Mbits/sec                  receiver
```

**HTTP 性能**

**相关工具**

**ab**
```java
 ./ab -n 10000 -c 2000 -p post/valid.txt -T application/json  http://localhost:8090/spring-test/valid
```

**wrk**
一个HTTP性能测试工具，内置了LuaJIT，根据实际需求，生成所需的请求负载，自定义响应的处理方法


## DNS

域名以分层的结构进行管理，域名解析也是递归方式（从顶级开始，以此类推）。发送给每个层级域名服务器。

DNS 服务通过资源记录的方式，来管理所有数据支持 A、CNAME、MX、NS、PTR 等多种类型的记录

- A 记录，用来把域名转换成 IP 地址；
- CNAME 记录，用来创建别名；
- NS 记录，则表示该域名对应的域名服务器地址。

**相关工具**

需按照 bind-utils 软件包

**nslookup**
查DNS信息

```java
[root@ecs-pre-business-01 ~]# nslookup www.car-house.cn
Server:		100.125.1.250 // 并不是直接管理的域名服务器，只是返回结果
Address:	100.125.1.250#53

Non-authoritative answer:
Name:	www.car-house.cn
Address: 114.116.239.62
```
加 -debug参数开启nslookup调试输出

有时候我们发现**DNS解析返回太慢**，我们可以**配置域名服务器**
```java
echo nameserver 114.114.114.114 > /etc/resolv.conf
```

**DNS解析不稳定**，我可以**使用 DNS 缓存**
```java
/etc/init.d/dnsmasq start

// 修改 /etc/resolv.conf，将 DNS 服务器改为 dnsmasq 的监听地址
echo nameserver 127.0.0.1 > /etc/resolv.conf

```


**dig**
提供了 trace 功能，可以展示递归查询的整个过程

```java
dig +trace +nodnssec www.car-house.cn
```

从上往下，四部分依次为根域名NS记录，顶级域名记录，二级域名记录，最终注解A记录

**tcpdump**

tcpdump基于libpcap，利用内核中的 AF_PACKET 套接字，抓取网络接口中传输的网络包

我们使用ping指令的时候，有时候发现 ping指令返回很慢，但是延迟却不高

可以通过tcpdump 在服务器中抓取和分析网络包

```java
$ tcpdump -nn udp port 53 or host 35.190.27.188
```

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/18.png)
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/19.png)

**Wireshark**

在Tcpdump得到的数据上进行分析的一个工具

## 缓解DDOS攻击

DDoS类型

- 耗尽带宽
- 耗尽操作系统的资源
- 消耗应用程序的运行资源

使用功能hpin3 作为攻击攻击
```java
// 客户端1发起攻击
hping3 -S -p 80 -i u10 192.168.0.30 // 效果不明显调小 u10,反之调大

// 服务器查看网络收包情况
sar -n DEV 1

// 服务器抓取具体网络包数据
tcpdump -i eth0 -n tcp port 80

09:15:48.287047 IP 192.168.0.2.27095 > 192.168.0.30: Flags [S], seq 1288268370, win 512, length 0
09:15:48.287050 IP 192.168.0.2.27131 > 192.168.0.30: Flags [S], seq 2084255254, win 512, length 0

```
通过观察 我们发现，大量的SYN包，极有可能是 SYN Flood 攻击。

- 客户端构造大量SYN包，请求建立TCP连接
- 服务受到连接请求后，向源IP发送SYN+ACK包，等待三次握手最后的客户端ACK报文，知道超时


**半开连接**

这种等待状态的TCP连接，称为半开连接，大量的半开连接导致连接表迅速占满。从而无法建立新的连接

```java
netstat -n -p | grep SYN_REC // 查看半开连接

netstat -n -p | grep SYN_REC | wc -l // 统计半开连接数

$ sysctl -w net.ipv4.tcp_max_syn_backlog=1024
net.ipv4.tcp_max_syn_backlog = 1024 // 修改半开连接容量

```
 DDoS 的分布式、大流量、难追踪等特点，目前还没有方法可以完全防御 DDoS 带来的问题，只能设法缓解这个影响
 
**DDOS攻击 应对方案**
```java
# 拒绝指定IP的包接收
iptables -I INPUT -s 192.168.0.2 -p tcp -j REJECT 

# 限制syn并发数为每秒1次
$ iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT

# 限制单个IP在60秒新建立的连接数为10
$ iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT

# 限制连接每个SYN_RECV失败后内核重试次数
$ sysctl -w net.ipv4.tcp_synack_retries=1
net.ipv4.tcp_synack_retries = 1

```
TCP SYN Cookies：SYN Cookies，防御SYN Flood攻击方法。基于连接信息（包括源地址、源端口、目的地址、目的端口等）以及一个加密种子（如系统启动时间），计算出一个哈希值（SHA1），这个哈希值称为 cookie。cookie 就被用作序列号，来应答 SYN+ACK 包，并释放连接状态。

这样不需要维护半开连接状态

```java
## 开启 SYN Cookies
sysctl -w net.ipv4.tcp_syncookies=1 // 配置都是临时的，重启后这些配置就会丢失.写入 /etc/sysctl.conf 文件才能持久化
net.ipv4.tcp_syncookies = 1 
```

DDoS 并不一定是因为大流量或者大 PPS，有时候，慢速的请求也会带来巨大的性能下降（这种情况称为**慢速 DDoS**）。**专业的流量清洗设备和网络防火墙**，在网络入口处**阻断恶意流量**，只保留正常流量进入数据中心的服务器


## 网络延迟

ping基于ICMP协议，计算**ICMP回显响应报文与ICMP 回显请求报文的时间差**，来获得往返延时

**获取网络延迟**

```java
hping3 -c 3 -S -p 80 baidu.com
traceroute --tcp -p 80 -n baidu.com
```

- TCP 延迟确认：针对 TCP ACK 的一种优化机制，不用每次请求都发送ACK，先等一会儿，看看有没有顺风车，若有，捎带ACK一起发送
- Nagle 算法（纳格算法）：TCP 协议中用于减少小包发送数量的一种优化算法，合并 TCP 小包，目的是为了提高实际带宽的利用率

TCP延迟确认和Nagle算法会导致延迟增长问题

## NAT
NAT技术通过重写IP数据包的源IP或目的IP，用来解决公网IP地址短缺的问题。

**NAT 分类**

- 静态NAT：内网IP和公网IP一对一永久映射关系
- 动态NAT：内网IP从公网IP池，动态选择一个进行映射
- NAPT：把内网IP映射到公网IP不同端口


**NAPT 分类**

- 源地址转换SNAT：目的地址不变，只替换源 IP 或源端口。常用于多个内网IP共享同一个公网IP
- 目的地址转换DNAT：源IP不变，只替换目的IP或目的端口。常用于通过公网IP不同端口号，访问内网多种服务，隐藏后端服务器真实IP地址
- 双向地址状态：同时使用DNAT和SNAT，先执行DNAT再执行SNAT

## 容器启动失败，慢

```java
dmesg命令查看系统日志
```

在docker等容器的使用中，没有资源限制，会导致容器占用整个资源，一旦程序发生异常，有可能拖垮整台机器

- 容器通过cgroups进行资源隔离，分析时要考虑 cgroups对应用程序的影响。
- 容器的文件系统、网络协议栈等跟主机隔离。虽然在容器外面，我们也可以分析容器的行为，不过有时候，进入容器的命名空间内部，可能更为方便。
- 容器的运行可能还会依赖于其他组件，比如各种网络插件、存储插件、设备插件等，让容器的性能分析更加复杂。如果你需要分析容器性能，别忘了考虑它们对性能的影响。

## 丢包

网络数据的收发过程，数据包还没传输到应用程序中，就被丢弃，被丢弃包的数量/总的传输包数，也就是我们常说的**丢包率**

丢包通常意味着网络拥塞和重传,导致网络延迟增大、吞吐降低

**丢包发生地**

- 两台 VM 连接之间，可能会发生传输失败的错误，比如**网络拥塞**、**线路错误**等；
- 在网卡收包后，**环形缓冲区**可能会因为溢出而丢包；
- 在链路层，可能会因为**网络帧校验失败**、QoS 等而丢包；
- 在 IP 层，可能会因为**路由失败、组包大小超过 MTU** 等而丢包；
- 在传输层，可能会因为**端口未监听、资源占用超过内核限制**等而丢包；
- 在套接字层，可能会因为**套接字缓冲区**溢出而丢包；
- 在应用层，可能会因为**应用程序异常**而丢包；
- 此外，如果配置了 **iptables** 规则，这些网络包也可能因为 iptables 过滤规则而丢包。

**链路层**
```java
## 显示网卡统计信息 可以查看丢包记录
netstat -i 
```

如果用tc等工具配置QoS，那么tc规则导致丢包，不会包含在网卡统计信息中

```java
## 检查eth0是否配置tc规则，查看有没有丢包 
tc -s qdisc show dev eth0
```

**网络层和传输层**

```java
## 查看各协议收发汇总，以及错误信息
netstat -s
```

**iptables**

```java
# 主机终端中查询内核配置  连接跟踪数
$ sysctl net.netfilter.nf_conntrack_max
net.netfilter.nf_conntrack_max = 262144
$ sysctl net.netfilter.nf_conntrack_count
net.netfilter.nf_conntrack_count = 182
```

```java
## 查看各条规则的统计信息
iptables -t filter -nvL
```
网卡的MTU设置过小也会导致丢包问题

## CPU使用率过高

**内核线程**

- ide进程为0号进程，系统创建**第一个进程**，初始1号，2号进程后，变为空闲任务。CPU上没有其他任务时，会运行它
- init进程为1号进程，通常是**systemd**进程，所有**用户态**进程的祖先
- kthreadd进程为2号进程，**内核态**运行，用来管理内核线程

```java
## 查找kthreadd进程的子进程，即内核线程
ps -f --ppid 2 -p 2
```

- ksoftirqd：处理软中断的内核线程，每个 CPU 上都有一个
- kswapd0：用于内存回收
- kworker：用于执行内核工作队列。分为绑定CPU和未绑定CPU
- migration：在负载均衡过程中，把进程迁移到 CPU 上
- bd2/sda1-8：为文件系统提供日志功能，以保证数据的完整
- pdflush：用于将内存中的脏页写入磁盘

```java
 ## 对指定进程进行调用栈查询并保存
 perf record -a -g -p 9 -- sleep 30
```

**火焰图**

- 横轴表示采样数和采样比例。一个函数占用横轴越宽，代表执行时间越长
- 纵轴表示调用栈，由下往上根据调用关系逐个展开。

```java
## 下载火焰图生成源码
git clone https://github.com/brendangregg/FlameGraph

cd FlameGraph  

1. 执行perf script，将perf record记录转换为可读采样记录
2. 执行stackcollapse-perf.pl脚本，合并调用栈信息；
3. 执行 flamegraph.pl 脚本，生成火焰图。
```

## 动态追踪

通过探针机制，采集内核或应用程序运行信息，获取丰富信息。

**DTrace**

运行常驻在内核中，用户可以通过dtrace命令，把D语
言编写的追踪脚本，提交到内核中的运行时来执行。

**SystemTap**

没有常驻内核的运行时，它需要先把脚本编译为内核模块，然后再插入到内核中执行

Dtrace 和 SystemTap 都会把用户传入的追踪处理
函数（一般称为Action），关联到被称为探针的检测点上。这些探针，实际上也就是各种动态追踪技术所依赖的事件源

**动态追踪的事件源**

- 静态探针：事先在代码中定义好，并编译到应用程序或者内核中的探针。如：tracepoints和USDT
- 动态探针：没有事先在代码中定义，但却可以在运行时动态添加的探针。如：内核态的 kprobes 和用于用户态的 uprobes
- 硬件事件：由性能监控计数器 PMC（Performance Monitoring Counter）产生，包括了各种硬件的性能情况，比如 CPU 的缓存、指令周期、分支预测等等

**动态追踪机制**

- ftrace
- perf
```java
## 查询所有支持事件
perf list

## 添加探针
perf probe --add do_sys_open

## 对 10s内do_sys_open函数采样
perf record -e probe:do_sys_open -aR sleep 10

## 追踪显示函数的参数
 perf probe -V do_sys_open
```
- eBPF
- BCC

## USE法

USE法把系统资源的性能指标，三个类别，使用率、饱和度以及错误数

- 使用率，资源用于服务的时间或容量百分比。100% 表示容量用尽或全部时间用于服务
- 饱和度，资源的繁忙程度，通常与等待队列的长度相关。100%表示资源无法接受更多的请求。
- 错误数表示发生错误事件个数。错误数越多系统的问题越严重。


![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/13.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/14.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/15.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/16.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/17.png)

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/18.png)
