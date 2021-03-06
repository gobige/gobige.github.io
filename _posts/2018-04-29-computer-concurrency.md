---
layout: post
title: '深入理解计算机系统-并发'
subtitle: '读书笔记系列深入理解计算机系统之并发'
date: 2018-04-29
categories: 计算机系统
author: yates
cover: ''
tags: 计算机系统
---

## 进程

### 程序计数器

多个进程并发时，多个逻辑程序计数器切换装载到物理程序计数器

### 进程的创建

- 系统初始化
- 系统调用：例：fork
- 用户请求调用：打开应用程序
- 批处理创建

### 进程的终止

- 正常退出：编译器完成编译，程序执行完成；用户点击关闭按钮
- 错误退出：用户引起错误，文件找不到
- 严重错误：不存在内存，非法计算
- 被其他进程杀死

### 进程的状态

- 运行：进程实际占用CPU时间片运行时
- 就绪：可运行，但因为其他进程正运行处于就绪状态
- 阻塞：除非外部事件（输入，资源）发生，否则进程不能运行

### 进程内容

每个进程都有一个地址空间，父进程和子进程有各自不同的地址空间，不可写的内存区域是共享的，可写的内存是不能被共享的


### 进程实现

通过维护一张进程表，包括程序计数器、堆栈指针、内存分配状况、所打开文件的状态、账号和调度信息，以及其他在进程由运行态转换到就绪态或阻塞态时所必须保存的信息，从而保证该进程随后能再次启动，就像从未被中断过一样

### 进程切换

硬件压入堆栈程序计数器等--》硬件从中断向量装入新的程序计数器--》汇编语言过程保存寄存器的值--》汇编语言过程设置新的堆栈--》
--》C 中断服务器运行（典型的读和缓存写入）--》调度器决定下面哪个程序先运行--》C 过程返回至汇编代码--》汇编语言过程开始运行新的当前进程

## 线程

线程会包含有一些进程的属性，所以线程被称为轻量的进程(lightweight processes)

### 为什么要有线程
- 多线程之间 共享 地址空间
- 创建线程比创建进程快 10 - 100 倍
- 多线程加速大量的I/O处理程序 吞吐量
 
### 线程的状态
和进程一样，多了终止状态

### 线程内容
独立的 程序计数器，寄存器，堆栈，状态

### 线程的，终止
线程调用库函数创建新线程，线程完成工作调用exit函数终止，还有join，yeild操作

POSIX 线程是可移植线程的标准实现。有如下系统调用：

- pthread_create	创建一个新线程
- pthread_exit	结束调用的线程
- pthread_join	等待一个特定的线程退出
- pthread_yield	释放 CPU 来运行另外一个线程
- pthread_attr_init	创建并初始化一个线程的属性结构
- pthread_attr_destory


### 线程实现
- 在用户空间中实现线程
	- 优势：
		- 线程切换时，线程信息保存在运行时环境提供线程表中，调度线程选择另一个线程。这两个过程都是**本地过程**。比内核调用效率高，不用切换到内核，**不需要上下文切换**，也不需要内存高速缓存flush。
		- 运行进程定制调度算法，可扩展性高。
	- 劣势：
		- 阻塞系统调用实现会影响其他进程中线程
		- 除非线程自愿的放弃CPU，否则其他线程都不能运行
- 在内核空间中实现线程
	- 优势：
		- 进程没有线程表，内核记录系统中所有线程的线程表，不需要运行时环境；
		- 阻塞调用通过系统调用方式实现。任何线程都有机会获取时间片
		- 缺页故障后，内核很容易的就能检查并切换线程
	- 劣势：
		- 创建或者销毁线程的开销比较大 
- 在用户和内核空间中混合实现线程
	- 自由控制用户线程和内核线程的数
	- 内核级线程会被多个用户级线程多路复用，内核只识别内核级线程，并对其进行调度

进程用于把资源集中在一起，而线程则是 CPU 上调度执行的实体

现代操作系统提供三种基本构造并发程序的方法

- 进程，每个逻辑流都是一个进程，由**内核调度**和维护，而且每个进程有**独立的虚拟地址空间**，如果进程间通信，控制流必须使用某种显式的进程间通信（IPC）机制
- I/O多路复用，应用程序在一个**进程上下文中显式调度自己逻辑流**，逻辑流模型化状态机，数据到达文件描述符后，主程序先是从一个状态切换为另外一个状态，因为在一个进程中，所以所有流**共享同一个地址空间**。
- 线程，线程运行在一个单一进程上下文中的逻辑流，**内核进行调度**,而且共享同一个虚拟地址空间

每个线程都有自己的线程上下文，包括唯一的整数线程ID，栈，栈指针，程序计数器，寄存器，条件码，同时共享统一进程下的整个虚拟地址空间

每个进程开始都是**单一线程**，称为**主线程**，某一时刻，主线程创建一个**对等线程**，cpu时间片在线程间切换，线程上下文切换**开销小于进程**上下文切换的，而且不像进程那样有严格的**父子层次**关系，所有跟一个进程有关的线程组成一个**对等线程池**，独立于其他线程创建的线程。一个线程可以**杀死**他任何的对等线程，或**等待**它的任意对等线程终止。对等线程读写相同**共享数据**

任何一个时间点上，线程是**可结合或可分离**的，一个可结合的线程能够被其他线程收回和杀死，分离的线程不能被其他现场回收或杀死，内存资源只有在它终止时系统自动释放。

## 进程间通信

### 三个问题

- 如何传递消息
- 如何保证线程之间不会干扰
- 数据先后问题

共享变量的互斥操作通常使用**信号量**来进行
- P（s）：如果s信号量非零。-1，并立即返回；如果为0则挂起
- V（s）：对s信号量加1，如果任何信号阻塞在P操作等待s变为非0，那么v重启这些线程中一个，完成P操作对s信号量-1

因为S信号量值总为0或1，这种依靠信号量保护共享变量的叫做二元信号量。P操作进行互斥锁加锁，V操作进行互斥锁解锁

**信号量**另外一个作用是调度共享资源的访问，比如生产者-消费者，读-写问题

**线程不安全函数**
- 不保护共享变量的函数，未保护全局变量的函数变成线程安全的，比较容易，使用类似P和V同步操作保护共享变量
- 保持跨越多个调用的状态函数，比如伪随机数生成器，这类想让其线程安全只有重写
- 返回指向静态变量的指针的函数，这类想让其线程安全**重写或加锁-复制技术（在每一个调用位置，对互斥锁加锁，将函数返回结果复制到一个私有的内存位置，然后解锁）**
- 调用线程不安全函数的函数，重写