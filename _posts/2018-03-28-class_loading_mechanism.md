---
layout: post
title: 'jvm之类加载机制'
subtitle: '类加载机制'
date: 2018-03-28
categories: jvm
author: yates
cover: ''
tags: jvm
---

# 前言
在java语言里面，jvm把类的数据从class文件加载到内存，并对数据进行校验，转换解析，初始化；而在类型的加载，连接，初始化过程都是在程序运行期间完成的，这种方式提高了应用程序的灵活性。

类的生命周期
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/35.png)
整个过程除了解析阶段，其他阶段都是相互交叉混合的执行，通常会在一个阶段执行过程中调用，激活另一个阶段

**必须立即对类进行“初始化”情况**
jvm规范规定下面几种情况必须立即对类进行初始化

- 遇到new，getstatic，putstatic，invokestatic这四条指令时，如果类未初始化则进行初始化；**代码场景**：new 关键字实例化对象；读取设置一个static字段（同时final，static修饰常量外，final修饰常量在编译阶段就把放入常量池中了），调用一个static修饰方法
- 使用反射对类进行反射调用的时候
- 对子类的初始化必先对父类进行初始化
- 虚拟机启动，执行程序入口方法类先初始化
- 当使用java动态特性，使用methodHandle实例进行方法入参调用时，代表的引用后面的类要先于使用前得到初始化

接口的初始化大致和类一致，在对父接口初始化的时候，不要求父接口全部完成初始化，只是在真正使用到了父接口的时候才会初始化。

### 加载
加载过程分为三个过程

- 通过一个类的全限定名获取定义此类的二进制字节流。
- 将这个字节流代表的静态存储结构转化为方法区运行时结构
- 在内存中生成一个代表这个类的class对象，作为方法区这个类各种数据的访问入口

第一步，获取二进制流可以通过很多方式进行获取，jar，war，ear包，网络，代理都可以获取

数组类型类加载时，如果组件类型是引用类型，则会使用该组件类型的类加载器进行加载；如果是基本类型，则会使用引导类加载器进行加载

### 验证
Java虚拟机只与Class文件这种特定的二进制文件格式所关联，使用java语言编写代码会在编译的时候对数组边界，类型转换，跳转等代码进行验证，但是如果是采用其他语言编写，编译成class文件就有可能没有经过一定的验证，造成载入系统奔溃的代码。

主要验证动作

- 文件格式验证，验证字节流是否符合class文件格式的规范，并且能被当前版本虚拟机处理。详见[java类文件结构](http://pev96mxgw.bkt.clouddn.com/2018/03/12/calss-structure.html)
- 元数据验证，对字节码描述的信息进行语义分析，使其符合java语言规范的要求，是否可继承，抽象类格式，接口格式等等。
- 字节码验证，通过数据流和控制流，分析程序的语义是否合法，符合逻辑，比如跳转区域，存取值类型。
- 符号引用验证，发生在虚拟机将符号引用转换为直接引用时，对自身类以外的信息进行匹配性校验（符号引用是否找得到对应类，引用字段，方法是否在对应类中有定义以及访问标志是否合法）

### 准备
主要是为类变量分配内存并初始化值，这里指的初始化是指基本类型的初始化，都是该数据类型零值。（final修饰常量为指定的常量值）

### 解析
主要是将常量池中**符号引用替换为直接引用**。
- 符号引用，在类文件结构中存储在表中，描述所引用的目标，跟虚拟机实现的内存布局无关
- 直接引用：跟虚拟机实现的内存布局有关，引用的目标必定已在内存中。

虚拟机实现根据需求判断是在类加载时进行解析，还是等到被使用前才进行解析。
解析动作主要针对类或接口，字段，类方法，接口方法，方法类型，方法句柄和调用点限符进行符号引用。

**类或接口的解析**
以当前代码所处D类中，要把一个从未解析过的符号N解析为一个类或接口C的直接引用。
- 如果C不是数组类型，把N的全限定类型传递给D的类加载器去加载类C，加载期间，会经历前面所有的元数据验证，字节码验证，父类加载等一些列动作。
- 如果数组C是一个数组类型，而且组件类型非基本类型，那么在经过和前面一样的规则加载组件类型的类，然后由虚拟机生成一个代表此数组维度和元素的数组对象。
- 经过上面的步骤，C在虚拟机中已经是一个有效的接口或类了，接下来符号引用验证访问权限

**字段解析**
字段解析会经过对**字段所属的类或接口**进行解析，得到C，完成后进行下面步骤：

- 如果C本身包含简单名称和字段描述符和解析字段相匹配，则直接返回这个字段直接引用。
- 上述若不成立，如果在C中实现了接口，则会按照继承关系从下往上递归搜索各个接口和父接口，直到找到简单名称和字段描述符和解析字段相匹配字段，返回这个字段直接引用。
- 如果接口查找不到，则按照继承关系从下往上递归搜索其父类
- 若还是没有找到，则抛出异常。

同样会对返回引用进行权限验证。

**如果同名字段同时出现在C的接口，父类中或同时在自己和父类多个接口中出现，编译器会拒绝编译**

**类方法解析**
和字段解析一样，需要先解析出方法所属类或接口的符号引用，解析完成，得到C，然后进行下面步骤：

- 类方法和接口方法的符号引用的常量类型定义不一样的，先对C引用进行验证是否是一个类引用
- 通过第一步后，在类C中查找是否有简单名称和描述符相匹配的方法，有则返回直接引用
- 若在类C中没有找到匹配方法，则在父类中递归查找具有匹配的方法，有则返回直接引用
- 若在父类中没有找到匹配方法，则在类C实现的接口列表和这些接口的父接口递归查找，如果有，说明类C是一个抽象类，抛出AbstractMethodError异常
- 否则，抛出noSuchMethodError异常

当然最后会对该方法引用进行权限验证

**接口方法解析**
和类方法解析一样，先解析出方法所属类或接口的符号引用，解析完成，得到C，然后进行下面步骤：

- 和类方法解析第一步一样，不过是对C引用验证是否是一个接口引用
- 通过第一步后，在接口C中查找是否有简单名称和描述符相匹配的方法，有则返回直接引用
- 若在类C中没有找到匹配方法，则在接口C的父接口递归查找，查找是否有简单名称和描述符相匹配的方法，如果有，说明返回方法直接引用。
- 否则，抛出noSuchMethodError异常

### 初始化
在初始化阶段，才真正执行类中定义的java程序代码，用另外一种角度表达，这个阶段是执行类构造器<clinit>方法的过程。

- <clinit>方法收集类中所有**类变量赋值动作和静态语句块中语句**合并产生
- 收集顺序取决于语句在源文件中出现的顺序，static代码块中对于在它之后定义的变量，可以复制，但是不能访问。
- <clinit>方法和<init>方法一样都会先初始化相应的父类方法
- 如果是接口<clinit>方法的话，不需要先执行父接口的<clinit>方法，只有当父接口的变量使用时，父接口才会初始化，接口实现类在初始化也不会执行接口<clinit>方法。
- 虚拟机会保证<clinit>方法只被一个线程执行一次，且只被执行一次

##类加载器
类加载器主要实现了在类加载阶段，**通过一个类的全限定名来获取描述此类的二进制字节流**，这个实现动作是放到java虚拟机外部去实现的。

每一个类，都需要由加载它的类加载器和这个类本身一同确立其在java虚拟机中的唯一性。

**双亲委派模型**

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/36.png)
从java虚拟机角度来看，只存在两种不同类加载器:

- 一种是**启动类加载器**，是虚拟机本身一部分；
- 另一种独立于虚拟机外部，用于加载其他所有类加载器，全部继承抽象类java.lang.classloader

从开发角度来看，可分为以下三种：

- 启动类加载器：负责加载java_home下**lib目录或-Xbootclasspath参数所指定路径中的类库**到虚拟机内存中，在开发中返回null可将加载请求委派给引导类加载器
- 扩展类加载器：由sun.misc.Launcher的ExtClassLoader实现，负责加载javahome下**lib\ext目录中或java.ext.dirs系统变量指定路径的类库**，该加载器在开发中可被直接使用
- 应用程序类加载器：由sun.misc.Launcher的App-ClassLoader实现，通过classLoader的getsystemclassloader方法得到，也被称为系统类加载器。负责**加载用户类路径上类库**

双亲委派模型工作过程是如果一个类加载器收到类加载请求，首先不会自己去尝试加载这个类，而是将加载请求**委派给父类加载器**，如果父类无法加载该请求，才自己去尝试加载。

双亲委派模型确保了java程序的稳定运作，避免了自定义类和基础类由于全限定名的一样导致的关系错乱，保证了java类型体系的行为。

双亲委派模型破坏，程序动态性，osgi，jndi
