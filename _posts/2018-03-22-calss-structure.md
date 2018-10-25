---
layout: post
title: 'class类文件结构'
subtitle: 'class类文件结构'
date: 2018-03-22
categories: jvm
author: yates
cover: ''
tags: jvm
---

# 前言
我们都知道java是一种基于jvm编译执行的语言，java文件编译成class文件，jvm只与Class文件这种特定的二进制文件格式所关联，任何能够编译成class文件结构的语言都可在jvm上运行如Scala，Groovy，Clojure等。

通常一个Class类文件都对应一个类和接口的定义信息，但是类和接口不一定定义在class文件里，也可以通过类加载器直接生成

class文件是由字节为基础单位的二进制流，各个数据项严格按照顺序紧凑排列在class文件中，顺序，数量，字节序都被严格按照位置存储

class文件格式如图

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/17.png)

**magic（魔数）和版本**
class文件头4个字节为魔数，作为jvm进行身份识别的标志，为16进制的OxCAFEBABE；第5个和第6个字节是次版本号，第7和第8是主版本号，在这里低版本的jvm会拒绝执行超过版本号的class文件，这也是为什么每个新版本的jdk兼容以前版本jdk原因

**常量池**

- **常量池容量计数值**，从1开始计数，表示定义的常量的数量。

常量池中存放两大类常量

- 字面量:文本字符串，声明为final的常量值等
- 符号引用：类和接口的全限定名，字段名称和描述符，方法名称和描述符（用于jvm加载class时进行动态连接获取内存入口）
- 常量池每一项常量都是一张表，如图
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/18.png) 
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/19.png) 
每种常量通过标志位来代表偏移量，每种常量的表结构都不一样

javap工具可以输出class文件中的常量表信息


**访问标志**
访问标志主要描述了类的一些信息

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/20.png)

**类索引，父类索引，接口索引**
主要用来确定类的继承关系，表示该类继承的类和实现的接口
- 除了object外，所有java类父类索引所有都不为0
- 首先通过constant_class_info找到索引，然后通过constant_UFT8_info表获取全限定名
- 类索引和父类索引都是2字节表示，接口索引时2字节数组表示

**字段表**
用于描述接口或类中声明的变量，不包含局部变量
如图
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/21.png)

- 访问标志位
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/22.png)

- 变量名引用，指向变量的常量命名，
- 描述符引用（字段数据类型）
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/23.png)
- 属性表集合，存储一些额外的信息

字段表不会列出从父类继承来的字段，但可能会列出原本java中不存在的字段，如内部类对外部类实例的字段

**方法表**
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/24.png)
可以看到基本和字段表是差不多的结构，除了属性表集合和访问标志的不同

- 方法中的**执行代码**，通过编译器生成字节码指令，存放到属性表集合中一个名为code的属性里忙
- java中无法仅仅通过返回值的不同来进行方法重载，但是class文件格式中，特征签名范围更大一些，只要描述符不一样，两个方法都可以合法共存

**属性表**
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/25.png)
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/26.png)

- code属性（code）
![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/27.png)
    - attribute_name_index：属性值名称索引
    - attribute_lentgh属性长度
    - max_stack代表了操作数栈深度最大值
    - max_locals局部变量表所需存储空间（以slot为基础单位对应32位bit，可重用）
    - code_length代表字节码长度
    - code存储字节码指令一系列字节流，每个字节码u1类型，可存储256条指令，已定义200多条指令
    - 异常表
      ![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/28.png)
如果当字节码在start_pc行和第end_pc行之间出现了类型catch_type或者子类的异常，则转到handler_pc行继续处理，当catch_type值为0时，代表任意清苦都要转到handler_pc处处理

java虚拟机执行字节码是基于栈的体系结构

- 异常属性（exceptions）

异常属性是个属性表同一级的属性，主要作用是列举出方法中可能抛出的受检查异常

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/29.png)
number_of_exceptions项表示方法可能抛出异常数量；exception_index_table表示每一种受检查异常，指向常量池汇总的索引

- linenumbertable属性

用于描述java源码行号与字节码行号之间关系。我们平常的日志中抛异常时出现的行号，以及debug时的断点位置都来源此

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/30.png)
line_number_table_length表示line_nubmer_info数量，line_nubmer_info含有start_pc和line_number数据项，分别表示字节码行号和java源码行号

- localVariableTable属性

用于描述栈帧中局部变量表中变量和java源码中定义的变量之间的关系，我们通过ide编写代码是方法的参数名称就来源于此

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/31.png)

local_ variable_ info中的start_pc，length数据项，分别表示局部变量生命周期开始的字节码偏移量和作用范围长度；name_ index 和 descriptor_ index表示局部变量的名称的常量索引和局部变量描述符；index表示局部变量在栈帧局部变量表中slot位置

LocalVariableTypeTable是泛型引入后用来作为泛型类型的特征说明

- sourcefile属性
用于记录生成这个class文件源码文件名称。sourceFile属性表中有sourceFile_index数据项指向源码文件名中常量池索引

- ConstantValue属性
通知jvm自动为静态变量赋值。如果同时使用final和static修辞变量，而且变量数据类型时基本类型或string的话，就生成constantvalue属性进行初始化；表中含有constantvalue_index数据项表示常量池中一个字面量的引用。

- InnerClasses属性
用于记录内部类和外部类之间的关联；表中包含number_of_classes和inner_classes数据项，分别表示内部类数量和具体内部类索引

![此处输入图片的描述](http://pev96mxgw.bkt.clouddn.com/img/2018-03-19-jvm/30.png)
inner_ class_ info_ index 和 outer_ class_ info_ index分别表示内部类和外部类的符号引用
inner_ name_ index和inner_ class_ access_ flags表示内部类名称索引，内部类访问标志

- Deprecated及Synthetic属性
这两个属性都是布尔类型的属性，只存在或不存在，前者表示某个类，方法，字段被表示不推荐使用；后者表示该字段或方法不是java源码直接产生而是由编译器添加

- StackMapTable属性
用于在jdk1.6之后的字节码验证阶段新类型验证器

- Signature属性
用于表示被泛型修辞的类，变量和方法，弥补java因为擦除带来的运行期无法获取到泛型信息

- BootstrapMethods属性
用于保存动态指令引用的引导方法限定符，如果常量池中出现CONSTANT_ InvokeDynamic_ info类型常量，那么这个类文件中必须存在一个该属性