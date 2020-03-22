---
layout: post
title: 'java并发'
subtitle: 'java并发的一些知识'
date: 2017-10-27
categories: java并发
author: yates
cover: 'http://cctv.com'
tags: java并发
---

### 一些概念

**同步，异步**
同步方法在被调用后，后面的代码只有在同步方法返回后才能执行；而异步方法不用等待返回，后面的代码就可以执行

**并行，并发**
并发：多个任务在同一个cpu核心中交替进行
并行：多个任务在多个cpu核心中一起进行

**线程优先级**
线程执行需要获得**cpu时间片**资源，线程优先级就是决定线程需要多或者少分配一些处理器资源的线程属性。在java中通过整形变量1~10设置，越大优先级越高（有些操作系统会忽略线程优先级设定）

**守护线程**
守护一些系统服务的线程

**线程安全**
指的是一个任务在多线程下执行的结果和在单线程下得到的结果是始终是一样的

**新建一个线程的方式**
1. 继承Thread类，重写run方法（不建议，java不支持多继承）
2. 实现runable接口
3. 实现callable接口
3. 使用Java的线程池

new Thread，线程进入新建状态，调用start方法，启动线程，使线程进入就绪状态，当分配到时间片后就开始运行了。而直接调用run方法，只是在当前线程下执行普通方法，并不是并发

线程A构造方法，静态块是有new A线程的线程B调用的，而run方法时线程A本身调用的

**CAS**

全称compare and set，是实现乐观锁的核心概念，通过内存共享变量实际值V，共享变量预期旧值O，共享变量修改后预期新值E，若V = O,则说明该共享变量没有被修改过，可以用E去替换O，若V != O说明变量已被修改过，重新刷新共享变量值，再次重试CAS操作直到成功或阻塞或中断；

但是CAS会面临ABA的问题;长期自旋开销；只能保证一个共享变量的执行操作

**线程状态转换**

- new：初始状态，线程被构建，还没调用start（）方法
- runnable：运行状态，**就绪**和**运行**统称运行中
- blocked：阻塞状态，线程获取资源受阻
- waiting：等待状态，线程进入等待状态，需要其他线程通知
- time_waiting；超时等待，在waiting指定时间后自行返回
- terminated：终止状态，当前线程已经执行完毕

**线程上下文切换优化**

1. 减少锁持有时间，Synchronized 同步锁资源，不仅带来线程上下文切换，还可能会增加进程间上下文切换
2. 降低锁粒度，通过锁分离和锁分段来避免所有线程对一个锁资源竞争过于激烈
3. 乐观锁 
4. wait/notifyAll导致过早唤醒其他未满足需求线程，最好指定唤醒
5. 合理设置线程池大小，避免创建过多线程
6. 使用协程实现非阻塞等待
7. 减少垃圾回收


**java中线程状态操作**

- interrupted：中断操作，表示一个运行中的线程是否被其他线程进行中断操作。在java中其他线程调用被中断线程的interrupt()方法进行中断操作（也可调用Thread的静态方法interrupted()对当前线程进行中断操作，同时**中断标志位**会被清除）
- join：线程A等待线程B线程终止后线程A才继续执行（或者设置等待超时后继续执行）
- sleep：让当前线程按照指定时间进行休眠，**让出cpu资源**，休眠期间**不会释放锁**，休眠完成后进入**就绪状态**
- yield：让当前线程**让出cpu资源**，进入就绪状态，继续和当前线程**相同优先级的线程**竞争cpu时间片
- wait：使一个线程处于等待（阻塞）状态，并且释放所持有的对象的锁



### 线程之间交互
线程通信方式主要有共享内存和线程通信方式两种方式，java采用的是共享内存的方式

**java的内存模型（JMM）**

java运行时数据区域如下

- 程序计数器-当前线程执行的字节码行号指示器
- Java虚拟机栈-java方法执行内存模型，包含局部变量表，操作数栈，动态链接，方法出口
- 本地方法栈：native方法用到的栈
- Java堆-**线程共享**的一块儿区域，存放java对象
- 方法区-加载类信息，常量，静态变量，即使编译代码
- 运行时常量池（方法区一部分）-编译期生成的各种字面量，符号
- 直接内存-NIO存放缓存

线程共享变量获取

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/1.png)

CPU的处理速度和主存的读写速度不是一个量级的，为了平衡这种巨大的差距，每个CPU都会有缓存，通常是三级缓存，L1,L2,L3;

- **L1**是最接近CPU的，**容量最小，速度最快，每个核上有两个L1Cache，一个存数据 L1d Cache，一个存指令 L1i Cache**)。
- **L2** Cache大一些，速度要慢一些，一般情况下每个核上都有**一个独立的L2Cache**；
- **L3**Cache是三级缓存中最大的一级，同时也是最慢的一级，在同一个CPU插槽之间的核**共享一个L3Cache**

线程A和线程B要完成通信，如下

1. 线程A从主内存中将共享变量读入线程A的工作内存后并进行操作，之后将数据重新写回到主内存中
2. 线程B从主存中读取最新的共享变量

如果在线程A还没有把数据刷入主内存时，线程B拿到主内存数据，并且使用，得到的数据就是脏数据，就发生了常说的线程安全问题

**重排序**
在不改变程序执行结果的前提下，cpu和编译器会对代码编译后的指令和代码进行重排序，从而提高执行效率

单线程环境，重排序不能改变程序运行结果；多线程环境下，会破坏多线程执行语义


代码执行过程
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/2.png)

**happens-before规则**
指的是两个操作之间的**执行顺序**，一个操作位于另一个操作之前发生，这两个操作可以同一个线程，也可以不是同一个线程。
JMM承诺保证happend-before的关系，不仅为程序员提供了足够强的**内存可见性**，而且对编译器和处理器的限制尽可能放松（允许不会改变程序结果的**重排序**）


- 定义 
    1. 如果一个操作happened-before另一个操作，那么第一个操作的执行结果对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前
    2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）

- 具体规则
    1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
    2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
    3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
    4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
    5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
    6. join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回
    7. 程序中断规则：对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
    8. 对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于发生它的finalize()方法的开始。

as-if-serial语义保证**单线程**内程序的**执行结果不被改变**，happens-before关系保证正确同步的**多线程**程序的**执行结果不被改变**

### 三大性质
- 原子性 一个操作不可中断，要么全部成功，要么全部失败
- 有序性 操作执行顺序不可变
- 可见性 可见性是指当一个线程修改了共享变量后，其他线程能够立即得知这个修改

**java中8种具有原子性操作**

- lock(锁定)：作用于主内存中的变量，它把一个变量标识为一个线程独占的状态；
- unlock(解锁):作用于主内存中的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便后面的load动作使用；
- load（载入）：作用于工作内存中的变量，它把read操作从主内存中得到的变量值放入工作内存中的变量副本
- use（使用）：作用于工作内存中的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；
- assign（赋值）：作用于工作内存中的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作；
- store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送给主内存中以便随后的write操作使用；
- write（操作）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

**注意，JMM规定了上述操作的限制**

- java内存模型只是要求上述两个操作是顺序执行的并不是连续执行的
- monitorenter和monitorexit指令对应lock和unlock操作，更上一层是synchronized满足操作原子性
- read和load操作必须顺序执行；
- 不允许read和load，store和write操作中其中一个单独出现；
- 不允许线程丢弃最近assign操作，即变量在工作内存改变后必须同步回主内存；
- 不允许线程无assign操作把数据从工作内存刷回主内存
- 一个新变量只能在主内存中诞生，不允许工作内存中直接使用未被初始化的变量
- 一个变量同一时刻只允许一条线程对其lock，但可被同一条线程多次lock，反之unlock也要多次
- 如果变量被lock后，执行引擎使用改变量前必须重新load或assign操作
- unlock前必须lock操作
- unlock前，必须先把变量同步回主内存，即执行store和write操作

### 并发的基础 synchronized
synchronized关键字可以使用在方法，**代码块**上，应用在**实例方法**和**静态方法**分别锁的是该类的**实例对象**和**类对象**（尽管new多个实例，但他们仍然是同一个类锁住）

**synchronized底层实现**

Synchronized是**JVM**实现的一种**内置锁**，锁的获取和释放是由JVM**隐式实现**。基于底层操作系统的**Mutex Lock**实现，每次获取和释放操作都会带来**用户态和内核态切换**

通常Synchronized实现同步锁的方式有两种，一种是修饰**方法**，一种是修饰**方法块**。前者使用了**ACC_SYNCHRONIZED访问标志**来区分一个方法是否是同步方法；后者是由**monitorenter和monitorexit指令**来实现同步的；都是对**Monitor对象**作持有和释放操作

**synchronized流程**
多个线程同时访问时，先被存放在**EntryList** 集合，设置**block状态**，线程获取到对象的Monitor（底层操作系统的Mutex Lock）后，**持有Mutex**；若线程调用**wait()**方法，**释放Mutex**，线程会进入**WaitSet集合**，等待被唤醒
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/performance/5.png)

java对象实例在堆内存被分为三个部分：

- 对象头（32位系统 8字节对象头，4字节引用地址；64位系统 16字节对象头，8字节引用地址；-XX:+UseCompressedOops（1.6默认开启） 开启压缩 64位系统对象也为12字节）
	- Mark Word：记录了对象和锁的信息
	- 指向类指针
	- 数组长度
- 实例数据
- 对齐填充

**Mark Word**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/performance/6.png)

Synchronized同步锁就是从**偏向锁**开始的，随着竞争越来越激烈，偏向锁升级到**轻量级锁**，最终升级到**重量级锁**

- **偏向锁**：用来优化**同一线程多次申请同一个锁**的竞争。当一个线程再次访问这个同步代码或方法时，该线程只需去对象头的MarkWord中去判断一下是否有**偏向锁指向它的ID**，无需再进入Monitor去竞争对象了。一旦出现**其它线程竞争锁**资源时，偏向锁就会被**撤销**。偏向锁的撤销需要等待全局**安全点**，暂停持有该锁的线程，同时检查该线程**是否还在执行**该方法，如果是，则**升级锁**，反之则被**其它线程抢占**

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/3.png) 

高并发，大量线程竞争同一个锁时，偏向锁的撤销，stop world会带来更大的性能消耗

- **轻量级锁**：适用于线程交替执行同步块
- **自旋锁**：轻量级锁CAS抢锁失败线程进入阻塞状态，如果锁的释放很快，那又要申请锁资源。JDK1.7默认启用自旋锁，但自旋次数不宜过多，因为会长时间占用CPU
- **重量级锁**：
```java
-XX:-UseBiasedLocking // 关闭偏向锁
-XX:-UseSpinning //参数关闭自旋锁优化(默认打开)
-XX:PreBlockSpin //参数修改默认的自旋次数。JDK1.7后，去掉此参数，由jvm控制
-XX:+UseHeavyMonitors  //设置重量级锁
```

### 死锁

产生条件

1. 互斥条件
2. 请求与保持条件
3. 不剥夺条件
4. 循环等待条件


### 乐观锁

悲观锁在高并发的场景下，激烈的锁竞争会造成线程阻塞，大量阻塞线程会导致系统的上下文切换，增加系统的性能开销。
乐观锁相比悲观锁来说，不会带来**死锁、饥饿**等活性故障问题，没有因竞争造成的系统开销。

**机制**

CAS是实现乐观锁的核心算法，它包含了3个参数：V（需要更新的变量）、E（预期值）和N（最新值）

**CPU层原子操作**

- 总线锁定：处理器要操作一个共享变量，总线上会发出一个Lock信号，其它处理器就不能操作该变量。但是会导致大量阻塞。
- 缓存锁定：处理器对缓存中的共享变量进行了操作，通知其它处理器放弃存储该共享资源或者重新读取该共享资源

**LongAdder**

长时间不断重试CAS，CPU会有非常大的执行开销。LongAdder通过**降低**操作共享变量的**并发数**，将变量的操作压力**分散到多个变量**值（变量数量不能超过CPU核数），每个写线程的value值分散到一个数组，不同线程命中到数组不同槽中
最后在读取值的时候会将原子操作的共享变量与各个分散在数组的value值相加，返回一个**近似准确**的数值，**最终返回值**为一个**准确的值**


### 动态编译锁消除/锁粗化

- **锁消除**：JIT编译器动态编译同步块时，借助于逃逸分析，判断同步块使用锁对象是否**只被一个线程访问**，如果确认JIT在编译时不会生成synchronized标志
- **锁粗化**：JIT动态编译时，如果相邻同步块使用同一个锁实例则会**合并几个同步块**，避免**反复申请，释放锁**带来的性能开销
- **锁粒度减小**：锁对象是数组或队列时，因为竞争激励可能会升级为重量级锁，**考虑将一个数组和队列对象拆成多个小对象，来降低锁竞争，提升并行度**，如ConcurrentHashMap。


### 并发的基础 volatile
1. 禁止cpu对指令进行重排序
2. 该变量对**所有线程可见性**。即变量值改变后，该变量会重新刷新到主内存，而其他所有拥有该变量的线程，都会重新去主内存刷新获取该变量值
	- 工作内存中的volatile变量在use操作时,会从主内存重新读取一遍volatile变量的值
	- 将当前处理器缓存行的数据写回系统内存时,会lock对应主内存中的volatile变量；

对于实现可见性，java还有final和synchronized可以实现


在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令

如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存，同时实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了

适用场景

- 运算结果并不依赖于变量的当前值
- 或者能够确保只有一个线程修改变量的值；
- 变量不需要与其他的状态变量共同参与不变约束


**long和double类型变量**
- long和double的非原子协定：对于这两种变量，运行虚拟机将没有被volatile修饰的64为数据读写操作划分为两次32位操作进行，因此这两种变量不用专门声明为volatile

### final
java中变量，可以分为**成员变量**以及方法**局部变量**

- final类变量：必须要在**静态初始化块**中指定初始值或者**声明**该类变量时指定初始值，而且只能在这两个地方之一进行指定；
- final实例变量：必须要在**非静态初始化块**，**声明**该实例变量或者在**构造器**中指定初始值，而且只能在这三个地方进行指定。
- final局部变量:由程序员进行**显式**初始化，如果final局部变量已经进行了初始化则后面就不能再次进行更改
- final修饰**基本数据类型**变量时，基本数据类型变量**不能重新赋值**，因此基本数据类型变量不能被改变;而对于**引用类型**变量而言，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的**地址不会发生改变**，即一直引用这个对象，但这个对象属性是可以改变的
- final方法:父类的final修辞方法,子类**不能重写**的,但是是**可以重载**的
- final类:一个类被final修饰后，该类是**不能继承**的
- 八个**包装类**和String类都是**不可变**类
- final域:**禁止**对final域的**写重排序**到构造函数之外,在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域就不具有这个保障
- 在一个线程中，**初次读对象引用**和**初次读该对象包含的final域**，JMM会禁止这两个操作的重排序
- 针对引用数据类型，final域写针对编译器和处理器重排序增加了这样的约束：在构造函数内对一个final修饰的对象的成员域的写入，与随后在构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的
- 在构造函数，不能让这个被构造的对象被其他线程可见，也就是说该对象引用不能在构造函数中“逸出”

### 漫谈juc
lock：类似于1.5之前的synchronize，显示获取和释放锁。基本操作是通过乐观锁来实现，但由于Lock锁也会在阻塞时被挂起，因此它依然属于悲观锁

**lock和synchronize的区别**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/4.png)

在并发量不高，竞争不激烈情况下，Synchronized同步锁通过**分级锁优势**，性能和Lock差不多；但**高负载，高并发**，Synchronized升级为**重量锁**，没有Lock稳定

**Lock接口定义的方法**

```java
void lock();
void lockInterruptibly();
boolean tryLock();
boolean tryLock(long time, TimeUnit unit);
void unlock();
Condition newCondition();
```

**Sync**
```java
private final Sync sync;
```

而Sync是继承的AbstractQueuedSynchronizer类，也就是我们常说的AQS.

**AQS同步器**
AQS同步器是用来构建锁和其他同步组件的基础框架，依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成同步队列

**同步队列**
当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列，AQS的同步队列通过链表实现

**同步组件**
所有同步组件的实现都依赖于aqs，需要继承AQS的静态内部类，重写aqs的提供的可重写方法，调用模板方法；aqs只负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，并提供getState(),setState(),compareAndSetState()方法修改同步状态

**aqs可重写方法**
```java
    protected int tryAcquireShared(int arg); // 共享式获取同步状态
	protected boolean tryAcquire(int arg);// 独占式获取同步状态
	protected boolean tryRelease(int arg);// 独占式释放同步状态
	protected boolean tryReleaseShared(int arg);// 共享式释放同步状态
	protected boolean isHeldExclusively();当前同步器是否在独占模式下被线程占用
```

**aqs模板方法**
```java
    public final void acquire(int arg); // 独占式获取同步状态
	public final void acquireInterruptibly(int arg);// 独占式获取同步状态,响应中断
	public final boolean tryAcquireNanos(int arg, long nanosTimeout);// 独占式获取同步状态,响应中断，增加超时效果
	public final void acquireShared(int arg);// 共享式获取同步状态
	public final void acquireSharedInterruptibly(int arg);// 共享式获取同步状态,响应中断
	public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout);// 共享式获取同步状态,响应中断,增加超时效果
	public final boolean release(int arg);// 独占式释放同步状态
	public final boolean releaseShared(int arg);// 共享式释放同步状态
	public final Collection<Thread> getQueuedThreads(); // 查询同步队列中等待线程集合
	
```

**实现者的角度**

- 通过可重写的方法,告诉AQS怎样判断当前同步状态是否成功获取或者是否成功释放。同步组件专注于对当前同步状态的逻辑判断，从而实现自己的同步语义。

**AQS的角度**

- 而对AQS来说，只需要同步组件返回的true和f-alse即可，因为AQS会对true和false会有不同的操作，true会认为当前线程获取同步组件成功直接返回，而false的话就AQS也会将当前线程插入同步队列


我们接下来看aqs的源码，来深入理解一下

AQS的**队列**实现由一个Node静态内部类
```java
static final class Node {
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;

        static final int CANCELLED =  1;// 节点从同步队列中取消
        static final int SIGNAL    = -1;// 后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行
        static final int CONDITION = -2;// 当前节点进入等待队列中
      
        static final int PROPAGATE = -3;// 表示下一次共享式同步状态获取将会无条件传播下去

		volatile int waitStatus;// 节点状态
		volatile Node prev;// 前置节点
		volatile Node next// 后置节点
		volatile Thread thread;// 当前加入队列线程节点
		Node nextWaiter;// 当前队列下一执行节点
		
		private transient volatile Node head;// 头指针
		private transient volatile Node tail;// 尾指针
		
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

- 节点的数据结构，即AQS的静态内部类Node,节点的等待状态等信息；
- 同步队列是一个双向队列，AQS通过持有头尾指针管理同步队列；



**同步器中的独占锁**
AQS提供了一个acquire方法来获取独占锁

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

通过我们重写的tryacqure方法获取独占锁，如果失败将当前线程加入同步队列

```java
private Node addWaiter(Node mode) {
	Node node = new Node(Thread.currentThread(), mode);
	// 判断当前同步队列尾节点是否为null
	Node pred = tail;
	if (pred != null) {
		// 将该节点的前置节点设为当前尾节点，进行cas校验，成功加入同步队列，设置尾节点的后置节点为当前节点
		node.prev = pred;
		if (compareAndSetTail(pred, node)) {
			pred.next = node;
			return node;
		}
	}
	enq(node);// 尾节点为null
	return node;
}


private final boolean compareAndSetHead(Node update) {
	return unsafe.compareAndSwapObject(this, headOffset, null, update);
}

private final boolean compareAndSetTail(Node expect, Node update) {
	return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}


private Node enq(final Node node) {
	// 这里会如果CAS失败会进行不断的自旋操作
    for (;;) {
        Node t = tail;
        if (t == null) { 
			// 将当前节点进行cas校验，成功加入同步队列，设置首尾节点
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
			// 将该节点的前置节点设为当前尾节点，进行cas校验，成功加入同步队列，设置尾节点的后置节点为当前节点
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

```

加入队列后的线程又怎么获取独占锁呢？继续来看

```java
final boolean acquireQueued(final Node node, int arg) {
	boolean failed = true;
	try {
		boolean interrupted = false;
		// 这里相当于每一个处于队列当中的节点都不断的进行自旋操作
		for (;;) {
			// 获取该节点的前置节点
			final Node p = node.predecessor();
			// 该节点前置节点是不是头节点而且获取独占锁成功
			if (p == head && tryAcquire(arg)) {
				// 设置当前节点为头节点
				setHead(node);
				// 设置下一个节点为null，便于jvm回收当前节点
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			// 获取锁失败，将节点设置为signal，表示当前线程阻塞
			if (shouldParkAfterFailedAcquire(p, node) &&
				parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}


private void setHead(Node node) {
	head = node;
	node.thread = null;
	node.prev = null;
}


private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	int ws = pred.waitStatus;
	if (ws == Node.SIGNAL)
		return true;
	if (ws > 0) {
		do {
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
		pred.next = node;
	} else {
		// 使用CAS将节点状态由INITIAL设置成SIGNAL，表示当前线程阻塞
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}




private final boolean parkAndCheckInterrupt() {
	LockSupport.park(this);
	return Thread.interrupted();
}
	

private void cancelAcquire(Node node) {
	if (node == null)
		return;

	node.thread = null;

	Node pred = node.prev;
	while (pred.waitStatus > 0)
		node.prev = pred = pred.prev;

	Node predNext = pred.next;

	node.waitStatus = Node.CANCELLED;

	if (node == tail && compareAndSetTail(node, pred)) {
		compareAndSetNext(pred, predNext, null);
	} else {
		int ws;
		if (pred != head &&
			((ws = pred.waitStatus) == Node.SIGNAL ||
			 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
			pred.thread != null) {
			Node next = node.next;
			if (next != null && next.waitStatus <= 0)
				compareAndSetNext(pred, predNext, next);
		} else {
			unparkSuccessor(node);
		}

		node.next = node; // help GC
	}
}

```

现在我们的线程拿到锁了，并且执行完锁内操作，那么该释放锁了

```java
public final boolean release(int arg) {
	// 释放独占锁成功
	if (tryRelease(arg)) {
		Node h = head;
		if (h != null && h.waitStatus != 0)
			unparkSuccessor(h);
		return true;
	}
	return false;
}

 private void unparkSuccessor(Node node) {
		// 
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        // 释放该节点后面所有不为null的节点的阻塞状态
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

**可中断式获取锁**
```java
public final void acquireInterruptibly(int arg)
		throws InterruptedException {
	if (Thread.interrupted())
		throw new InterruptedException();
	if (!tryAcquire(arg))
		doAcquireInterruptibly(arg);
}

private void doAcquireInterruptibly(int arg)
	throws InterruptedException {
	final Node node = addWaiter(Node.EXCLUSIVE);
	boolean failed = true;
	try {
		for (;;) {
			final Node p = node.predecessor();
			if (p == head && tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return;
			}
			
			// 不同于普通获取锁，节点被阻塞，直接中断代码，抛出异常
			if (shouldParkAfterFailedAcquire(p, node) &&
				parkAndCheckInterrupt())
				throw new InterruptedException();
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```


**超时等待式获取锁（tryAcquireNanos()方法**

```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
		throws InterruptedException {
	if (Thread.interrupted())
		throw new InterruptedException();
	return tryAcquire(arg) ||
		doAcquireNanos(arg, nanosTimeout);
}

private boolean doAcquireNanos(int arg, long nanosTimeout)
		throws InterruptedException {
	if (nanosTimeout <= 0L)
		return false;
		// 计算超时时长
	final long deadline = System.nanoTime() + nanosTimeout;
	final Node node = addWaiter(Node.EXCLUSIVE);
	boolean failed = true;
	try {
		for (;;) {
			final Node p = node.predecessor();
			if (p == head && tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return true;
			}
			nanosTimeout = deadline - System.nanoTime();
			if (nanosTimeout <= 0L)
				return false;
				
				// 线程阻塞
			if (shouldParkAfterFailedAcquire(p, node) &&
				nanosTimeout > spinForTimeoutThreshold)
				LockSupport.parkNanos(this, nanosTimeout);
				// 若线程中断则抛出异常
			if (Thread.interrupted())
				throw new InterruptedException();
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

**共享锁的获取，释放**
```java
// 共享锁获取
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
					// 当该节点的前驱节点是头结点且成功获取同步状态
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}


// 共享锁释放
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

看完了AQS的实现，我们来看依赖AQS实现的一些具体的锁类

### ReentrantLock

**锁获取过程**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/5.png)

ReentrantLock **重入锁**，支持重入性，表示能够对共享资源重复加锁，当前线程获取该锁再次获取不会被阻塞（线程重入，计数+1；释放锁时，也要释放重入的次数）(降低上下文切换，降低性能开销，保证系统最大的吞吐量)
```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //1. 如果该锁未被任何线程占有，该锁能被当前线程获取
	if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	//2.若被占有，检查占有线程是否是当前线程
    else if (current == getExclusiveOwnerThread()) {
		// 3. 再次获取，计数加一
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

protected final boolean tryRelease(int releases) {
	//1. 同步状态减1
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
		//2. 只有当同步状态为0时，锁成功被释放，返回true
        free = true;
        setExclusiveOwnerThread(null);
    }
	// 3. 锁未被完全释放，返回false
    setState(c);
    return free;
}
```

ReentrantLock通过构造方法也可支持**公平锁和非公平锁**(区别是对待同步队列的获取锁处理方式不一样)，默认是不公平锁，公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程(降低上下文切换)或刚进入线程获取到锁（降低性能开销，保证系统最大的吞吐量）

```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

static final class NonfairSync extends Sync {
	private static final long serialVersionUID = 7316153563782823691L;

	final void lock() {
		if (compareAndSetState(0, 1))
			setExclusiveOwnerThread(Thread.currentThread());
		else
			acquire(1);
	}

	protected final boolean tryAcquire(int acquires) {
		return nonfairTryAcquire(acquires);
	}
	
	final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
}


    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }


public final boolean hasQueuedPredecessors() {
	// The correctness of this depends on head being initialized
	// before tail and on head.next being accurate if the current
	// thread is first in queue.
	Node t = tail; // Read fields in reverse initialization order
	Node h = head;
	Node s;
	return h != t &&
		((s = h.next) == null || s.thread != Thread.currentThread());
}
```





### ReentrantReadWriteLock

**读写锁**

读写锁内部维护了两个锁，一个是用于读操作的ReadLock，一个是用于写操作的WriteLock

1. 允许同一时间被多个读线程访问，但在写线程访问时，所有读线程和写线程会被阻塞;支持可重入性.
2. 同时支持锁降级,当一个线程获取到写锁后,又获取读锁,在释放写锁,那么自动降级为读锁

读写锁自定义同步器（继承AQS）在同步状态state上使用**高16位表示读，低16位表示写**,维护多个读线程和一个写线程的状态。

**获取写锁过程**

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/6.png)

**获取读锁过程**

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/7.png)


读写锁实现了ReadWriteLock接口
```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}

// 写锁的获取
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
	// 1. 获取写锁当前的同步状态
    int c = getState();
	// 2. 获取写锁获取的次数，将同步状态与1左移16位做与运算
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
		// 3.1 当读锁已被读线程获取或者当前线程不是已经获取写锁的线程的话
		// 当前线程获取写锁失败
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
		// 3.2 当前线程获取写锁，支持可重复加锁
        setState(c + acquires);
        return true;
    }
	// 3.3 写锁未被任何线程获取，当前线程可获取写锁
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}

static final int SHARED_SHIFT   = 16;
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }


// 写锁的释放
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
	//1. 同步状态减去写状态
    int nextc = getState() - releases;
	//2. 当前写状态是否为0，为0则释放写锁
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
	//3. 不为0则更新同步状态
    setState(nextc);
    return free;
}

// 读锁的获取
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
	//1. 如果写锁已经被获取并且获取写锁的线程不是当前线程的话，当前
	// 线程获取读锁失败返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
		//2. 当前线程获取读锁
        compareAndSetState(c, c + SHARED_UNIT)) {
		//3. 下面的代码主要是新增的一些功能，比如getReadHoldCount()方法
		//返回当前获取读锁的次数
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
	//4. 处理在第二步中CAS操作失败的自旋已经实现重入性
    return fullTryAcquireShared(current);
}

// 读锁的释放
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
	// 前面还是为了实现getReadHoldCount等新功能
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
		// 读锁释放 将同步状态减去读状态即可
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}

// 锁降级  遵循按照获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁
void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // Must release read lock before acquiring write lock
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // Recheck state because another thread might have
                // acquired write lock and changed state before we did.
                if (!cacheValid) {
                    data = ...
            cacheValid = true;
          }
          // Downgrade by acquiring read lock before releasing write lock
          rwl.readLock().lock();
        } finally {
          rwl.writeLock().unlock(); // Unlock write, still hold read
        }
      }
 
      try {
        use(data);
      } finally {
        rwl.readLock().unlock();
      }
    }
}
```

### StampedLock

读取很多、写入很少的情况下，ReentrantReadWriteLock会造成**写进程饥饿问题**（写进程因为迟迟无法获取到锁一直等待）

StampedLock控制锁有三种模式: 写、悲观读以及乐观读。

- 获取锁时会返回一个**票据stamp**，获取的stamp除了在释放锁时需要校验。
- 乐观读模式下，stamp还会作为读取共享资源后的二次校验


### 使用LockSupport 阻塞和唤醒线程

```java
lockSupport提供的方法
void park() 阻塞线程  //synchronzed致使线程阻塞，线程会进入到BLOCKED状态，而调用LockSupprt方法阻塞线程会致使线程进入到WAITING状态
void parkNanos(long nanos) 超时阻塞线程
void parkUntil(long deadline) 超时阻塞线程
void unpark(Thread thread):  唤醒线程
```

### condition 语言级别的等待/通知机制

condition相较于比较底层的wait和notify方式，具有

- 支持中断操作
- 支持多个等待队列
- 支持超时设置

condition适合lock配合一起使用，lock.newcondition(),该方法会new 出一个ConditionObject对象，也就是AQS的一个内部类，condition内部也会维持一个**单向的等待队列**；lock因为配合condition使用，所以一个lock除了拥有一个同步队列，还可以拥有多个等待队列


condition.await 使当前获得锁的线程进入等待队列
```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
	// 1. 将当前线程包装成Node，尾插入到等待队列中
    Node node = addConditionWaiter();
	// 2. 释放当前线程所占用的lock，在释放的过程中会唤醒同步队列中的下一个节点
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
		// 3. 当前线程进入到等待状态
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
	// 4. 自旋等待获取到同步状态（即获取到lock）
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
	// 5. 处理被中断的情况
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
	//将当前线程包装成Node
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
		//尾插入
        t.nextWaiter = node;
	//更新lastWaiter
    lastWaiter = node;
    return node;
}


final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        if (release(savedState)) {
			//成功释放同步状态
            failed = false;
            return savedState;
        } else {
			//不成功释放同步状态抛出异常
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}


while (!isOnSyncQueue(node)) {
	// 3. 当前线程进入到等待状态
    LockSupport.park(this);
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}
```
整个过程是调用**condition.await()**方法后，会使得**当前线程释放lock**然后**加入到等待队列中**，直至被**signal/signalAll**后会使得当前线程从**等待队列中移至到同步队列**中去，直到**获得了lock后才会从await方法返回**，或者在等待时被中断会做**中断处理**

**退出等待状态**

1. 逻辑走到break退出while循环（当前**线程被中断**）；
2. while循环中的逻辑判断为false（当前处于等待队列的线程被**移动到同步队列）

**等待队列的唤醒**
```java
public final void signal() {
    //1. 先检测当前线程是否已经获取lock
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    //2. 获取等待队列中第一个节点，之后的操作都是针对这个节点
	Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}


private void doSignal(Node first) {
    do {
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
		//1. 将头结点从等待队列中移除
        first.nextWaiter = null;
		//2. while中transferForSignal方法对头结点做真正的处理
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}

final boolean transferForSignal(Node node) {
	//2.将该节点移入到同步队列中去
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}

```


### ConcurrentHashmap
**HashMap获得线程安全的实现方式**

- 使用Hashtable
- 对hashMap对象加锁 synchronize关键字
- Collections.synchronizedMap() 其实现也是使用synchronize关键字对象加锁
- concurrenthashmap 性能，线程安全都能保证，推荐

JDK1.7中，ConcurrentHashMap使用了分段锁Segment减小了锁粒度,提高了并发度


在理解concurrentHashmap之前我希望你看过hashmap的源码，因为concurrenthashmap大体的结构，插入去除思想跟hashMap是差不多的，这里我只会解析不同的地方 [HashMap源码解析入口](http://muyibeyond.cn/2018/06/01/source-dataStructrue-map.html)
**ConcurrentHashmap重要的变量**

```java

concurrenthashmap的Node元素内key对象和value对象,table,等变量都加了volitile关键字

nextTable volatile Node<K,V>[] nextTable; //扩容时使用，平时为null，只有在扩容的时候才为非null

sizeCtl volatile int sizeCtl; 标识table数组大小，**当值为负数时：**如果为-1表示正在初始化，如果为-N则表示当前正有N-1个线程进行扩容操作；**当值为正数时：**如果当前数组为null的话表示table在初始化过程中，不为null代表数组需容量

sun.misc.Unsafe U 在ConcurrentHashMapde的实现中可以看到大量的U.compareAndSwapXXXX的方法去修改ConcurrentHashMap的一些属性。这些方法实际上是利用了CAS算法保证了线程安全性

````

**几个重要的方法**
```java
 // 该方法用来获取table数组中索引为i的Node元素。
 static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
     return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
 }
 
 //利用cas设置索引为i的Node元素
 static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                     Node<K,V> c, Node<K,V> v) {
     return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
 }
 
 // 该方法用来设置table数组中索引为i的元素
  static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
     U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
 }
```

那么我们具体来看concurrenthashmap的实现方法
```java
// 初始化table方法
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
		// 通过判断是否等于-1
        if ((sc = sizeCtl) < 0)
			// 1. 保证只有一个线程正在进行初始化操作，让出cpu时间片，让已经开始初始化的线程进行操作
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
					// 2. 得出数组的大小
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
					// 3. 这里才真正的初始化数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
					// 4. 计算数组中可用的大小：实际大小n*0.75（加载因子）
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}

```

最重要的方法put方法来了
```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
	//1. 计算key的hash值，为了减少hash碰撞，使用的是spread()方法是一个比hashmap更复杂的hash算法
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
		//2. 如果当前table还没有初始化先调用initTable方法将tab进行初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
		//3. tab中索引为i的位置的元素为null，则直接使用CAS将值插入即可
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
			// 依靠的使用cas插入数值
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
		//4. 当前正在扩容，使用-1标识正在扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
			//  而在链表插入的时候使用的是锁，锁住当前数组元素以及后续所有next节点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
					//5. 当前为链表，在链表中插入新的键值对
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
					// 6.当前为红黑树，将新的键值对插入到红黑树中
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
			// 7.插入完键值对后再根据实际大小看是否需要转换成红黑树
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
	//8.对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容 
    addCount(1L, binCount);
    return null;
}
```

获取node元素get方法
```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
	// 1. 重hash
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 2. table[i]桶节点的key与查找的key相同，则直接返回
		if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
		// 3. 当前节点hash小于0说明为树节点，在红黑树中查找即可
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
		//4. 从链表中查找，查找到则返回该节点的value，否则就返回null即可
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

容器扩容
```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
	//1. 新建Node数组，容量为之前的两倍
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
	//2. 新建forwardingNode引用，在之后会用到
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 3. 确定遍历中的索引i
		while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
		//4.将原数组中的元素复制到新数组中去
		//4.5 for循环退出，扩容结束修改sizeCtl属性
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
		//4.1 当前数组中第i个元素为null，用CAS设置成特殊节点forwardingNode(可以理解成占位符)
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
		//4.2 如果遍历到ForwardingNode节点  说明这个点已经被处理过了 直接跳过  这里是控制并发扩容的核心
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
						//4.3 处理当前节点为链表的头结点的情况，构造两个链表，一个是原链表  另一个是原链表的反序排列
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                       //在nextTable的i位置上插入一个链表
                       setTabAt(nextTab, i, ln);
                       //在nextTable的i+n的位置上插入另一个链表
                       setTabAt(nextTab, i + n, hn);
                       //在table的i位置上插入forwardNode节点  表示已经处理过该节点
                       setTabAt(tab, i, fwd);
                       //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                       advance = true;
                    }
					//4.4 处理当前节点是TreeBin时的情况，操作和上面的类似
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

### ConcurrentLinkedQueue

concurrentLinkedQueue 是JUC提供的线程安全的队列


```java
单向链表的结构
private static class Node<E> {
	volatile E item;
	volatile Node<E> next;
	}

//  设置链表头节点和尾节点	
private transient volatile Node<E> head;
private transient volatile Node<E> tail;
	
```

node节点提供的安全的插入操作
```java
Node(E item) {
	UNSAFE.putObject(this, itemOffset, item);
}

//更改Node中的数据域item	
boolean casItem(E cmp, E val) {
    return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
}
//更改Node中的指针域next
void lazySetNext(Node<E> val) {
    UNSAFE.putOrderedObject(this, nextOffset, val);
}
//更改Node中的指针域next
boolean casNext(Node<E> cmp, Node<E> val) {
    return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
}
```

队列的增加元素（offer）是在链表尾进行和删除元素（poll）是在链表头进行，想想如果两个线程对其同时操作offset，poll；前者快，链表越来越长，后者快，链表越来越短；而在两者操作的交集区域（临界点）就会出现offset->poll->offset和poll->offset->poll问题


**offse源码**
```java
public boolean offer(E e) {
		// 是否为null进行判断
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);

        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
			// 当前没有元素插入，q是最后一个元素
            if (q == null) {
                // p is last node
                if (p.casNext(null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this queue,
                    // and for newNode to become "live".
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            else if (p == q)
                // We have fallen off list.  If tail is unchanged, it
                // will also be off-list, in which case we need to
                // jump to head, from which all live nodes are always
                // reachable.  Else the new tail is a better bet.
                p = (t != (t = tail)) ? t : head;
            else
                // Check for tail updates after two hops.
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
````
TODO 未完待续


### CopyOnWriteArrayList

Copy-On-Write(COW)，即写时复制的思想,当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器，是一种读写分离的思想，延时更新的策略

```java
/// get方法
public E get(int index) {
    return get(getArray(), index);
}
/**
 * Gets the array.  Non-private so as to also be accessible
 * from CopyOnWriteArraySet class.
 */
final Object[] getArray() {
    return array;
}
private E get(Object[] a, int index) {
    return (E) a[index];
}


// add方法
public boolean add(E e) {

    final ReentrantLock lock = this.lock;
	//1. 使用ReentrantLock独占锁,保证写线程在同一时刻只有一个
    lock.lock();
    try {
		//2. 获取旧数组引用
        Object[] elements = getArray();
        int len = elements.length;
		//3. 创建新的数组，并将旧数组的数据复制到新数组中
        Object[] newElements = Arrays.copyOf(elements, len + 1);
		//4. 往新数组中添加新的数据	        
		newElements[len] = e;
		//5. 将旧数组引用指向新的数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

读写锁和cow区别
- **读写锁**在获取到读锁时，写线程是不能操作的，反之，写锁获取的时候，读线程是要阻塞的，从而解决“脏读”等问题，而**cow**是牺牲实时性而保证数据最终一致性，读线程在写线程操作期间，对数据的获取时延时的

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/8.png)

### ThreadLocal
在线程安全方面除了用synchronize和lock来控制临界区资源，这种方式无疑都会造成线程的阻塞，而我们上面提到的cow思想，其实java也提供了threadLocal这个类

threadLocal通过以当前ThreadLocal为key，把value存放在ThreadLocalMap容器里面，而ThreadLocalMap是当前Thread的成员变量，如下

ThreadLocal的核心代码
```java
public void set(T value) {
	//1. 获取当前线程实例对象
    Thread t = Thread.currentThread();
	//2. 通过当前线程实例获取到ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
		//3. 如果Map不为null,则以当前threadLocl实例为key,值为value进行存入
        map.set(this, value);
    else
		//4.map为null,则新建ThreadLocalMap并存入value
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

ThreadLocal.ThreadLocalMap threadLocals = null;


// 获取threadLocal的值，即使不存在，初始化返回null
public T get() {
	//1. 获取当前线程的实例对象
    Thread t = Thread.currentThread();
	//2. 获取当前线程的threadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
		//3. 获取map中当前threadLocal实例为key的值的entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
			//4. 当前entitiy不为null的话，就返回相应的值value
            T result = (T)e.value;
            return result;
        }
    }
	//5. 若map为null或者entry为null的话通过该方法初始化，并返回该方法返回的value
    return setInitialValue();
}

// 新增一个key为threadLocal的value为null的
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}

// 移除一个key为threadLocal的元素
public void remove() {
	//1. 获取当前线程的threadLocalMap
	ThreadLocalMap m = getMap(Thread.currentThread());
 	if (m != null)
		//2. 从map中删除以当前threadLocal实例为key的键值对
		m.remove(this);
}


// ThreadLocalMap结构中的维护的一个元素entry变量用来装threadLocal的引用

private Entry[] table;

// 这里是一个弱引用
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

看了上面的代码可以大致分析出ThreadLocal的一个情况，当我们创建一个threadLocal的时候，内存会分配出一块区域给他，
- 当我们向它set值时候，它会去取当前thread的对象，并且创建（使用）thread的一个由弱引用entry数组结构为变量的map，以当前threadLocal为key，将set值填充进去；
- 当我们get ThreadLocal存进去的值时候，如果thread里面map没有这个key，则会创建相应entry设置vaue为null并返回
- 当移除threadLocal的时候会找到map中对应的entry并且将其引用为null

看了代码不明觉厉，为什么ThreadLocal会导致内存溢出的问题，比如：我们常见的线程池，线程用完后都会放入池中，不会回收，等待下一次连接使用，这样就不会频繁的线程创建，销毁，导致性能的消耗，但是线程中的threadLocal变量在使用后很多时候是没有remove的，里面的value对象在thread里面entry数组是还有引用的，根据jvm GC原理，具有可达性可能，而这部分只有threadLocal进行remove或者线程销毁后，失去引用才会回收

针对threadLocal会造成的内存泄漏问题,threadLocal也有一些措施，在set值时
```java
private void set(ThreadLocal<?> key, Object value) {

	Entry[] tab = table;
	int len = tab.length;
	int i = key.threadLocalHashCode & (len-1);

	for (Entry e = tab[i];
		 e != null;
		 e = tab[i = nextIndex(i, len)]) {
		ThreadLocal<?> k = e.get();

		if (k == key) {
			e.value = value;
			return;
		}

		if (k == null) {
			replaceStaleEntry(key, value, i);
			return;
		}
	}

	tab[i] = new Entry(key, value);
	int sz = ++size;
	if (!cleanSomeSlots(i, sz) && sz >= threshold)
		rehash();
}

//  遇到key为null的对象进行清理
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
							   int staleSlot) {
	Entry[] tab = table;
	int len = tab.length;
	Entry e;

	int slotToExpunge = staleSlot;
	for (int i = prevIndex(staleSlot, len);
		 (e = tab[i]) != null;
		 i = prevIndex(i, len))
		if (e.get() == null)
			slotToExpunge = i;

	for (int i = nextIndex(staleSlot, len);
		 (e = tab[i]) != null;
		 i = nextIndex(i, len)) {
		ThreadLocal<?> k = e.get();

		if (k == key) {
			e.value = value;

			tab[i] = tab[staleSlot];
			tab[staleSlot] = e;

			// Start expunge at preceding stale entry if it exists
			if (slotToExpunge == staleSlot)
				slotToExpunge = i;
			cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
			return;
		}

		if (k == null && slotToExpunge == staleSlot)
			slotToExpunge = i;
	}

	tab[staleSlot].value = null;
	tab[staleSlot] = new Entry(key, value);

	if (slotToExpunge != staleSlot)
		cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}

// cleanSomeSlots方法检测并清除脏entry
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```

**threadLocalMap的结构**

threadLocalMap和hashmap这些map不一样的地方是threadLocalMap只是普通的散列表，通过**开放定址法**解决hash冲突（threadLocal的hash算法分布式比较均匀的），而hashmap是使用**分离链表法**解决hash冲突

- 开发定址法 关键字散列到的数组单元已经被另外一个关键字占用的时候，就会尝试在数组中寻找其他的单元，直到找到一个空的单元


### BlockingQueue
当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止

```java
put：当阻塞队列容量已经满时，往阻塞队列插入数据的线程会被阻塞，直至阻塞队列已经有空余的容量可供使用；
offer(E e, long timeout, TimeUnit unit)：若阻塞队列已经满时，同样会阻塞插入数据的线程，直至阻塞队列已经有空余的地方，与put方法不同的是，该方法会有一个超时时间，若超过当前给定的超时时间，插入数据的线程会退出；
删除数据

take()：当阻塞队列为空时，获取队头数据的线程会被阻塞；
poll(long timeout, TimeUnit unit)：当阻塞队列为空时，获取数据的线程会被阻塞，另外，如果被阻塞的线程超过了给定的时长，该线程会退出
```

BlockingQueue的实现类

- ArrayBlockingQueue 数组实现的有界队列，创建后容量不可变，可作为有界数据缓冲区，队列满了插入数据和队列空了消费数据都会造成线程的阻塞，同样支持公平和非公平，公平的话会降低吞吐量
- LinkedBlockingQueue 链表实现的有界阻塞队列，默认容量等于Integer.MAX_VALUE
- PriorityBlockingQueue 支持优先级的无界阻塞队列，可以指定排序的规则，
- SynchronousQueue 每个插入操作都要等待另一个线程进行删除操作
- LinkedTransferQueue 链表结构的无界阻塞队列 
- LinkedBlockingDeque 链表结构有界双端队列
- DelayQueue 无界阻塞队列，只有当数据对象的延迟时间到达时才能入队

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/12.png)

### 线程池

**线程实现模型**

- **1:1线程模型（轻量级进程和内核线程一对一相互映射）**：KLT（内核线程）由操作系统内核支持线程，通过调度器对线程切换。Linux通过**fork()创建子进程代表内核线程**。新的进程被分配资源，然后**复制父进程所有值**到新
进程中，只有**PID等值不一致**。这种方式**冗余数据，浪费内存，消耗CPU时间**初始化数据。**LWP（轻量级进程）**LWP使用clone调用创建线程，**部分复制父进程资源**，剩余通过**指针共享**
- **N:1线程模型（用户线程和内核线程实现的N:1）**：在**用户空间完成线程创建，同步，销毁和调度**，不需要内核的帮助。避免用户态和内核态空间切换
- **N:M线程模型**：N:1线程模型的缺点在于操作系统不能感知用户态的线程，因此容易造成某一个线程进行系统调用内核线程时被阻塞，从而导致整个进程被阻塞。N:M线程模型支持用户态菜鸟仓通过LWP与内核线程连接，用户态的线程数量和内核态的LWP数量是N:M的映射关系

**协程**

可以把协程看作是一个**类函数或者一块函数中的代码**，一个线程可创建多个协程。协程使用N:M线程模型。适用于I/O密集型应用，避免上下文频繁切换

java可通过Kilim协程框架来实现协程


- Scheduler是Kilim实现协程的核心调度器，Scheduler负责分派Task给指定的工作者线程
- Mailbox对象类似一个邮箱，协程之间可以依靠邮箱来进行通信和数据共享，避免了线程安全问题
- Fiber是实现N:M线程映射的关键


**线程池作用**
1. 线程池的出现降低了线程创建和销毁所带来的消耗
2. 提升了连接后数据的响应速度
3. 管理了线程，避免无限制的创建线程


**Java提供线程池**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/9.png)


**线程池处理请求流程**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-10-27-javaconcurrent/10.png)

建议使用ThreadPoolExecutor控制线程数量

ThreadPoolExecutor构造方法入参
- corePoolSize 核心线程池大小
- maximumPoolSize 线程池最大容量
- keepAliveTime 空闲线程存活时长，特指除核心线程外线程
- unit 时间单位
- BlockingQueue<Runnable> workQueue 阻塞队列
- ThreadFactory threadFactory 创建线程工厂类
- RejectedExecutionHandler handler 线程饱和策略 
- shutdownNow 停止所有正在执行和未执行任务，并且返回未执行任务列表
- shutdown 中断所有没有执行任务
- isTerminated 所有线程是否都关闭成功

**线程池最大线程的配置**
- cpu密集型任务：cpu核心数量 + 1
- IO密集型任务：2 * cpu核心数量
- 需要按优先级执行任务：使用PriorityBlockingQueue队列
- 等待长时间返回结果任务：尽量多配制线程池
 
```java
线程数=N（CPU核数）*（1+WT（线程等待时间）/ST（线程时间运行时间））
```

JDK自带的工具VisualVM来查看WT/ST比例,根据自己的业务场景，从“N+1”和“2N”两个公式中选出一个适合的，计算出一个大概的线程数量，之后通过实际压测，逐渐往“增大线程数量”和“减小线程数量”这两个方向调整，然后观察整体的处理时间变化，最终确定一个具体的线程数量


阿姆达尔定律指出优化串行是优化系统性能的关键。我们生产中有多个不同类型任务要执行，应该分别创建线程池。过分批次依赖，建议拆开服务，分别部署

### futrueTask
- 获取任务执行结果的异步任务  具有未启动，启动中，已完成三个状态；
- 通过get获取任务状态，但是会阻塞当前线程，直到任务中断或结束，才会返回结果
- 通过cancel方法取消任务，未启动状态：任务永远不会执行；已启动状态：cancel(true) 中断线程，阻止任务执行，cancel(false)，中断还未执行任务；已完成状态：返回false


### JUC提供的原子操作类

这些原子操作类都是使用乐观锁CAS实现，有AtomicBoolean，AtomicInteger，AtomicLong，AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray

### CountDownLatch 和 CyclicBarrier

CountDownLatch设定一个计数器，当调用该对象计算器为0时，被countdownLltch阻塞的线程才会继续执行
- await() 调用该方法的线程等到CountDownLatch计算器为0时才继续执行
- await(long timeout, TimeUnit unit) 超时继续执行
- countDown()：使CountDownLatch计算器-1
- ong getCount()：获取当前CountDownLatch计算器值

CyclicBarrier

- await() 调用该方法的线程阻塞，直到CyclicBarrier要求的阻塞线程数量达到

- await(long timeout, TimeUnit unit) 超时继续执行

- int getNumberWaiting() 获取被阻塞的线程数量

- boolean isBroken() 查询阻塞等待的线程是否被中断

- void reset() 初始化屏障（再来一次）


**区别**

- CountDownLatch，CyclicBarrier都是强调等待的线程数达到某个计数值，然后返回继续wait方法处理流程
- CyclicBarrier提供方法更多，可在多个线程某个事件达成的时候执行barrierAction事件
- CountDownLatch是不能复用的，而CyclicLatch达成事件后又会继续重新，计数。是可以复用的


### Semaphore

Semaphore可以理解为信号量，可支持公平/非公平，经常拿来做流量控制
- void acquire() 当前线程获取semaphore信号量，Semaphore信号量-1，如果获取Semaphore信号量当前不足则阻塞等待直至能够获取为止

- void acquire(int permits) 获取多个信号量

- void release() 释放信号量

- void release(int permits) 释放多个信号量

- boolean tryAcquire() 获取信号量，获取成功返回ture，失败false

- boolean tryAcquire(int permits) 获取多个信号量，获取成功返回ture，失败false

- boolean tryAcquire(long timeout, TimeUnit unit) 在指定时间内获取信号量，获取成功返回ture，失败false

- boolean tryAcquire(int permits, long timeout, TimeUnit unit) 在指定时间内获取多个信号量，获取成功返回ture，失败false

- int availablePermits() 返回当前还剩余信号量数量

- int getQueueLength() 返回正在等待获取信号量的线程数

- boolean hasQueuedThreads() 是否有线程正在等待获取信号量

- Collection<Thread> getQueuedThreads() 获取所有正在等待信号量的线程集合


### Exchanger
用于两个线程共同到达某个事件时交换线程数据

- exchange(V x)  交换线程数据

- exchange(V x, long timeout, TimeUnit unit) 超时交换线程数据


