---
layout: post
title: 'java并发'
subtitle: 'java并发的一些知识'
date: 2017-10-27
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
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


**线程状态转换**
new：初始状态，线程被构建，还没调用start（）方法
runnable：运行状态，**就绪**和**运行**统称运行中
blocked：阻塞状态，线程获取资源受阻
waiting：等待状态，线程进入等待状态，需要其他线程通知
time_waiting；超时等待，在waiting指定时间后自行返回
terminated：终止状态，当前线程已经执行完毕

**java中线程状态操作**

- interrupted：中断操作，表示一个运行中的线程是否被其他线程进行中断操作。在java中其他线程调用被中断线程的interrupt()方法进行中断操作（也可调用Thread的静态方法interrupted()对当前线程进行中断操作，同时**中断标志位**会被清除）
- join：线程A等待线程B线程终止后线程A才继续执行（或者设置等待超时后继续执行）
- sleep：让当前线程按照指定时间进行休眠，**让出cpu资源**，休眠期间**不会释放锁**，休眠完成后进入**就绪状态**
- yield：让当前线程**让出cpu资源**，进入就绪状态，继续和当前线程**相同优先级的线程**竞争cpu时间片



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

[此处输入图片的描述](http://www.muyibeyond.cn/img/2017-10-27-javaconcurrent/1.png)

CPU的处理速度和主存的读写速度不是一个量级的，为了平衡这种巨大的差距，每个CPU都会有缓存，通常是三级缓存，L1,L2,L3;L1是最接近CPU的，容量最小，速度最快，每个核上有两个L1Cache，一个存数据 L1d Cache，一个存指令 L1i Cache)。L2 Cache大一些，速度要慢一些，一般情况下每个核上都有一个独立的L2Cache；L3Cache是三级缓存中最大的一级，同时也是最慢的一级，在同一个CPU插槽之间的核共享一个L3Cache

线程A和线程B要完成通信，如下

1. 线程A从主内存中将共享变量读入线程A的工作内存后并进行操作，之后将数据重新写回到主内存中
2. 线程B从主存中读取最新的共享变量

如果在线程A还没有把数据刷入主内存时，线程B拿到主内存数据，并且使用，得到的数据就是脏数据，就发生了常说的线程安全问题

**重排序**
在不改变程序执行结果的前提下，cpu和编译器会对代码编译后的指令和代码进行重排序，从而提高执行效率

代码执行过程
[此处输入图片的描述](http://www.muyibeyond.cn/img/2017-10-27-javaconcurrent/1.png)

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

- java内存模型只是要求上述两个操作是顺序执行的并不是连续执行的
- monitorenter和monitorexit指令对应lock和unlock操作，更上一层是synchronized满足操作原子性


### 并发的基础 synchronized
synchronized关键字可以使用在方法，代码块上，应用在实例方法和静态方法分别锁的是该类的实例对象和类对象（尽管new多个实例，但他们仍然是同一个类锁住）

**synchronized底层实现**
在java的锁机制中，在java或者类

### 并发的基础 volatile
1. 将当前处理器缓存行的数据写回系统内存；
2. 这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效

在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令

如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存，同时实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了

运算结果并不依赖于变量的当前值，或者能够确保只有一个线程修改变量的值；
变量不需要与其他的状态变量共同参与不变约束

### final
java中变量，可以分为**成员变量**以及方法**局部变量**

- 类变量：必须要在静态初始化块中指定初始值或者声明该类变量时指定初始值，而且只能在这两个地方之一进行指定；
- 实例变量：必要要在非静态初始化块，声明该实例变量或者在构造器中指定初始值，而且只能在这三个地方进行指定。
- final局部变量由程序员进行显式初始化，如果final局部变量已经进行了初始化则后面就不能再次进行更改
- final修饰基本数据类型变量时，不能对基本数据类型变量重新赋值，因此基本数据类型变量不能被改变。而对于引用类型变量而言，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的地址不会发生改变，即一直引用这个对象，但这个对象属性是可以改变的
-  父类的final方法是不能够被子类重写的
- final方法是可以被重载的
- 当一个类被final修饰时，表名该类是不能被子类继承的
- 八个包装类和String类都是不可变类
- 写final域的重排序规则禁止对final域的写重排序到构造函数之外,在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域就不具有这个保障
- 在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序
- 在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序
- 针对引用数据类型，final域写针对编译器和处理器重排序增加了这样的约束：在构造函数内对一个final修饰的对象的成员域的写入，与随后在构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的
- 在构造函数，不能让这个被构造的对象被其他线程可见，也就是说该对象引用不能在构造函数中“逸出”

### 漫谈juc
lock：类似于1.5之前的synchronize，失去了像synchronize关键字隐式加锁解锁的便捷性，但是拥有了锁获取和释放的可操作性；以及可中断的获取锁，超时获取锁等同步特性

**同步器aqs**
同步器是用来构建锁和其他同步组件的基础框架，依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成等待队列。

**同步队列**

当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列，AQS的同步队列通过链表实现

AQS的队列实现由一个Node静态内部类

```java
volatile int waitStatus;// 节点状态
volatile Node prev;// 前置节点
volatile Node next// 后置节点
volatile Thread thread;// 当前加入队列线程节点
Node nextWaiter;// 当前队列下一执行节点
```

**同步器中的独占锁**
AQS提供了一个acquire方法来获取独占锁

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```
调用acquire方法**成功，返回;失败，将当前线程加入同步队列**

```java
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }


  private final boolean compareAndSetHead(Node update) {
        return unsafe.compareAndSwapObject(this, headOffset, null, update);
    }

    /**
     * CAS tail field. Used only by enq.
     */
    private final boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }


private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}


    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
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
```
锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作



**同步组件**
同步组件的实现依赖于同步器AQS，在同步组件实现中，使用AQS的方式被推荐定义继承AQS的静态内存类；
AQS采用模板方法进行设计，AQS的protected修饰的方法需要由继承AQS的子类进行重写实现，当调用AQS的子类的方法时就会调用被重写的方法；
AQS负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，而Lock等同步组件主要专注于实现同步语义；
在重写AQS的方式时，使用AQS提供的getState(),setState(),compareAndSetState()方法进行修改同步状态

AQS提供的模板方法可以分为3类：

- 独占式获取与释放同步状态；
- 共享式获取与释放同步状态；
- 查询同步队列中等待线程情况；

同步组件实现者的角度：

- 通过可重写的方法：独占式：tryAcquire()(独占式获取同步状态），tryRelease()（独占式释放同步状态）；共享式：tryAcquireShared()(共享式获取同步状态)，tryReleaseShared()(共享式释放同步状态)；告诉AQS怎样判断当前同步状态是否成功获取或者是否成功释放。同步组件专注于对当前同步状态的逻辑判断，从而实现自己的同步语义。

AQS的角度

- 而对AQS来说，只需要同步组件返回的true和f-alse即可，因为AQS会对true和false会有不同的操作，true会认为当前线程获取同步组件成功直接返回，而false的话就AQS也会将当前线程插入同步队列


### ReentrantLock
ReentrantLock **重入锁**，支持重入性，表示能够对共享资源重复加锁，当前线程获取该锁再次获取不会被阻塞（线程重入，计数+1；释放锁时，也要释放重入的次数）；ReentrantLock通过构造方法也可支持**公平锁和非公平锁**，公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程能再次获取到锁（降低上下文切换，降低性能开销，保证系统最大的吞吐量）

### ReentrantReadWriteLock
**读写锁**允许同一时间被多个读线程访问，但在写线程访问时，所有读线程和写线程会被阻塞

TODO 未完待续
