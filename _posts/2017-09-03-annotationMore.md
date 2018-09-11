---
layout: post
title: '注解知多少'
subtitle: '关于java的注解你又知道多少呢'
date: 2017-09-03
categories: java底层
author: yates
cover: 'http://cctv.com'
tags: jdk
---

### 前言
本文将介绍注解的基本结构，语法；使用场景；怎么自定义注解；java中常见的注解等知识
	    
### 注解的基本语法
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/1.png)
    
- 注解使用@interface关键字定义
        
- @Target是这个注解的作用域，也就是说这个注解可以写在什么地方，有下列作用域范围
    * CONSTRUCTOR（构造方法声明）
    * FIELD（字段声明）
    * LOCAL VARIABLE（局部变量声明）
    * METHOD（方法声明）
    * PACKAGE（包声明）
    * PARAMETER（参数声明）
    * TYPE（类接口）
            
- @Retention是该注解的生命周期，就是说在Java程序从编译，解析到执行，什么时候能够获取得到该注解，有下列生命周期定义
    * SOURCE（只在源码显示，编译时丢弃）
    * CLASS（编译时记录到class中，运行时忽略）
    * RUNTIME（运行时存在，可以通过反射读取）
            
- @Inherited是一个标识性的元注解，它允许子注解继承它
        
-  @Documented，生成javadoc时会包含注解
        
- @Repeatable是可重复的意思。注解的值可以同时取多个
        
- 可以看到方法体内有name，values等成员变量（成员以无参无异常的方式声明，成员变量可以用default指定一个默认值的）
    * 成员类型是受限制的，合法的类型包括基本的数据类型以及String，Class，Annotation,Enumeration等
    * 如果注解只有一个成员，则成员名必须取名为value()，在使用时可以忽略成员名和赋值号（=）
    ![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/3.png)
    * 注解类可以没有成员，没有成员的注解称为标识注解。
            
- 使用注解的语法：@<注解名>(<成员名1>=<成员值1>,<成员名1>=<成员值1>,…)

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/2.png)

### Java中常见的注解
####@Override
我们都知道，java三大特性的继承中，方法是可以覆盖的，如图：

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/4.png)

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/5.png)

所有类都隐式从object继承，testAno类**目的是覆盖**object的equals方法,但是得到的结果却是错误的，因为我们**重写**object的equals方法时制定了一个非object类型参数，从而引发了这个方法**重载**，而不是重写的效果，当我们引入@Override注解后，会在编译的时候提示报错的信息，因此我么可以使用该注解来确保子类的方法也覆盖超类中的非最终具体方法或抽象方法以及接口方法
    
####@Deprecated
在开发代码时，有时候代码会变得过时，在新的调用中被其他更合适的代码替换，但是为了兼容以前的对该代码调用的地方的正常运行，以及在未来的版本中被弃用，可以使用该注解，在程序运行中使用过时代码的时候会得到warning提示

####@FunctionalInterface
随着jdk8的lambda表达式引入，函数式接口（一个函数式接口只有一个抽象方法，由于默认方法有一个实现，所以他们不是抽象的）越来越流行，这些接口可以使用lambda表达式，方法引用，或构造函数引用代替。

```java
public interface Foo {
    public int doSomething();
}

public interface Bar {
    public int doSomething();
    public default int doSomethingElse() {
        return 1;
    }
}
```

因此，下面的对象可以用lambda表达式代替

```java
class FunctionalConsumer {
    public void consumeFoo(Foo foo) {
        System.out.println(foo.doSomething());
    }
    public void consumeBar(Bar bar) {
        System.out.println(bar.doSomething());
    }
}

// lambda调用方法
FunctionalConsumer functionalConsumer = new FunctionalConsumer();
        functionalConsumer.consumeBar(() -> 10);
        functionalConsumer.consumeBar(() -> 20);
```

如果我们错误的将Foo和Bar接口定义为非函数式接口，那么编译器便不会报错直到运行报错，于是可以使用FunctionalInterface注解，编译器便指出错误

####@SuppressWarnings
警告是编译器组成部分，为开发人员提供反馈，可能的危险行为或在未来的编译器版本中可能会出现的错误，例如在java 泛型类型使用没有关联正式泛型参数。为了忽略某些特定警告，可以使用该注解

####@SafeVarargs
参数安全类型注解 
同样还有很多著名的第三方注解，如：spring的Autowired，Service等

###注解的提取
标注为生命周期在运行时状态的注解可以通过反射来提取，如下：

```java
@MyIF(author = "muyibeyond",desc = "test anotation class")
public class AnnotationTest {
	@MyIF(author = "muyibeyond",desc = "test anotation method")
	public void method() {
		System.out.println("do something");
	}

	public static void main(String[] args) {
		try {
			// 加载类
			Class c = Class.forName("practice.base.AnnotationTest");
			// 获取是否有对应注解
			boolean classAnoIsExist = c.isAnnotationPresent(MyIF.class);

			if (classAnoIsExist) {
				// 拿到注解实例，解析注解
				MyIF d = (MyIF) c.getAnnotation(MyIF.class);
				System.out.println("编写这个类的作者：" + d.author() + "类描述：" + d.desc());
			}

			// 获取类方法
			Method[] ms = c.getMethods();
			for (Method m : ms) {
				boolean meheodAnoIsExist = m.isAnnotationPresent(MyIF.class);
				if (meheodAnoIsExist) {
					MyIF d=m.getAnnotation(MyIF.class);
					System.out.println("编写这个方法的作者：" + d.author() + "类描述：" + d.desc());
				}
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}

@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface MyIF{
	String author() default "yates";
	String desc() default "";
}
```

运行结果：

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2017-09-03-annotationMore/6.png)

### 注解的使用场景
那么应该在什么地方使用注解呢？我们来看看官方的解释
注解是一系列***元数据***，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。
注解有许多用处，主要如下： 
- 提供信息给编译器： 编译器可以利用注解来**探测错误和警告信息** 
- 编译阶段时的处理： 软件工具可以用来利用注解信息来**生成代码、Html文档**或者做其它相应处理。 
- 运行时的处理： 某些注解可以在**程序运行**的时候**接受代码的提取**

说白了注解怎么使用取决于你想利用它干什么，通常注解主要给编译器及工具类型的软件用的，还有很重要的一点注解的提取需要借助于 Java 的**反射**技术，反射比较**慢**，所以注解使用时也需要谨慎计较**时间成本**。
