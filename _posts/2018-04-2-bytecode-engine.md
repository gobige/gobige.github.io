---
layout: post
title: 'jvm之虚拟机字节码执行引擎'
subtitle: '虚拟机字节码执行引擎'
date: 2018-04-02
categories: jvm
author: yates
cover: ''
tags: jvm
---

# 前言
物理机处理引擎是建立在处理器，硬件，指令集，和操作系统层面上，而虚拟机执行是自行指定指令集和执行引擎结构体系。

### 运行时栈帧结构
**栈帧**是虚拟机运行时数据区中虚拟机栈的元素，存储了方法局部变量表，操作数栈，动态连接和方法返回地址等信息，在程序编译的时候，栈帧的大小就已经确定好了；一个线程的方法调用栈很长，但是只有栈顶的栈帧才是活动中的。

**局部变量表**

- 局部变量表是一组变量值存储空间，存放方法参数和方法内部局部变量，以容量槽slot为最小单位，这个跟class类文件的slot有所不同，随处理器，操作系统的不同有不同定义，但是虚拟机规定能存下8种基本数据类型。
- 为了节省空间，slot在栈帧中是可以重用的，当字节码pc计数器值超过某个变量的作用域，这个变量对应的slot域就会被其他变量使用，这种重用的方式在某些情况下会影响gc的行为。对于在方法中占用大量内存，方法的栈帧长时间不能回收的可以使用变量=null的赋值来使其回收。
- 因为局部变量不存在类变量的准备阶段，所以局部变量未赋初值会导致编译报错

**操作数栈**
操作数栈的每一个元素可以是任意的java数据类型，方法运行期，各种字节码指令往操作数栈中写入和提取内容。
操作数栈中元素的数据类型必须与字节码指令序列严格匹配

**动态链接**
每个栈帧都包含一个指向运行时常量池的引用，从而支持方法在调用过程中的动态链接

**方法返回地址**
退出一个方法执行有两种方式，一种方式是方法执行过程中遇到返回的字节码指令，通过把返回值传递给上层方法调用者来结束在调用方法内的执行过程，另一种在调用方法执行过程中遇到异常，而且这个异常没有得到处理，从而异常退出的方式，这种方式不会给上层调用者返回任何值。


### 方法调用
方法调用的目的是确定被调用的方法版本。

**解析调用**
虚拟机在类加载过程中对方法调用有一部分是在解析阶段将符号引用转化为直接引用，而其他的则是在运行期间才会得到最终的目标方法的直接引用。前者大多是私有方法,静态方法,构造器，父类方法，final修饰方法这些在运行期前就能确定的调用版本。详情见-> [jvm类加载机制](http://muyibeyond.cn/2018/03/28/class_loading_mechanism.html)

**分派调用**

- **静态分派** 在java的类继承关系中，声明父类获取子类对象这是被允许的，我们在很多代码中常见用父类作为方法入参类型，而实际方法body内实现的是子类的调用，通过父类调用子类这个过程叫做多态；我们把方法入参的父类叫做**静态类型**，而虚拟机在选择方法调用的时候会**根据静态类型去选择调用重载的方法**，这种调用称为**静态分派**;

```java
public class StaticDispatch {
    static abstract class Human {
    }

    static class Man extends Human {
    }

    static class Woman extends Human {
    }

    public void sayHello(Human guy) {
        System.out.println(" hello, guy！");
    }

    public void sayHello(Man guy) {
        System.out.println(" hello, gentleman！");
    }

    public void sayHello(Woman guy) {
        System.out.println(" hello, lady！");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch sr = new StaticDispatch();
        sr.sayHello(man);
        sr.sayHello(woman);
    }
}
```
```java
hello, guy！
hello, guy！
```

而在java的世界里，重载版本可能不是唯一的，只能是**最合适**的

```java
public class Overload {
    public static void sayHello(Object arg) {
        System.out.println(" hello Object");
    }

    public static void sayHello(int arg) {
        System.out.println(" hello int");
    }

    public static void sayHello(long arg) {
        System.out.println(" hello long");
    }

    public static void sayHello(Character arg) {
        System.out.println(" hello Character");
    }

    public static void sayHello(char arg) {
        System.out.println(" hello char");
    }

    public static void sayHello(char… arg) {
        System.out.println(" hello char……");
    }

    public static void sayHello(Serializable arg) {
        System.out.println(" hello Serializable");
    }

    public static void main(String[] args) {
        sayHello('a');
    }
}
```

输出结果很明显会是hello char，但是如果依次从大到小入参去掉对应的方法，会出现不一样的输出结果。这个过程中会发生自动类型转换，装箱，查找父类接口等操作

- **动态分派**
```java
public class DynamicDispatch {
    static abstract class Human {
        protected abstract void sayHello();
    }

    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println(" man say hello");
        }
    }

    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println(" woman say hello");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        man.sayHello();
        woman.sayHello();
        man = new Woman();
        man.sayHello();
    }
}
```

输出结果

```java
man say hello 
woman say hello
woman say hello
```

整个查找最终执行方法过程如下

- 找到引用所指向的对象实际类型C
- 如果在C中找到常量中描述符合简单名称都相符的方法，则进行权限校验，返回方法的直接引用
- 如果在第二步没有找到相应方法，则根据继承关系由下向上查找C的父类进行第二步的搜索校验
- 如果第三步还是没找到就抛出abstractMethodError异常

上述代码是我们很常见的**方法重写**，我们把这样的方法调用方式成为**动态分派**

**单分派，多分派**

```java
public class Dispatch {
    static class QQ {
    }

    static class _360 {
    }

    public static class Father {
        public void hardChoice(QQ arg) {
            System.out.println(" father choose qq");
        }

        public void hardChoice(_360 arg) {
            System.out.println(" father choose _360");
        }
    }

    public static class Son extends Father {
        public void hardChoice(QQ arg) {
            System.out.println(" son choose qq");
        }

        public void hardChoice(_360 arg) {
            System.out.println(" son choose _360");
        }
    }

    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();
        father.hardChoice(new _360 ());
        son.hardChoice(new QQ());
    }
}
```

```java
father choose 360 
son choose qq
```

方法的接受者和方法的参数统称方法的宗量
上面选择执行方法过程如下：
- 首先在编译阶段方法入参有QQ和360，接受类型有father和son，根据两个宗量的选择，选择father.hardchoice(360)和father.hardchoice(qq)，这个过程是**多分派**
- 然后在运行期，虚拟机不关心入参的类型，只关心方法接受者的实际类型是father还是son，而作为一个宗量作为选择，这个过程称为**单分派**

综上所述，可以得到今天的java是一门静态多分派，动态单分派的语言

运行时动态分派需要在类的方法元数据中搜索合适的目标方法，基于性能的考虑，java虚拟机在方法区中建立一个虚方法表，来代替元数据的查找提高性能

**动态语言类型**

- 特征：类型检查在运行期而不是编译器；变量无类型而变量值才有类型
- 优点：实现功能更加清晰和简洁，提高开发的效率
- 在编译期无法提供严谨的类型检查，不利于代码的稳定性达到更大规模

**基于寄存器的指令集**
java编译器输出的指令流，是一种基于栈的指令集架构，还有一种基于寄存器的指令集，最典型的就是现在大多数主流pc使用的X86指令集，大致区别如下：

- 基于栈的指令集主要优点是可移植，避免了硬件的约束，当然执行速度会相对寄存器指令集会慢一些；基于栈的指令集代码相对紧凑一些，编译器实现更加简单，不需要考虑空间分配的问题，但是完成相同的功能所需的指令数量会比寄存器架构多，而且栈实现始终是在内存中，频繁的栈访问也就意味着频繁的内存访问，内存会成为执行速度的瓶颈。


