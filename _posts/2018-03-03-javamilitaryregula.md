---
layout: post
title: 'java编程规范'
subtitle: '一些搜集java开发规范'
date: 2017-03-03
categories: 规范
author: yates
cover: 'http://cctv.com'
tags: java
---

### 核心军规

- 进行比较运算时，把字符串常量放在前面

- 任何集合检查null和长度

- 除了接口（专门用于继承）所有的方法都应该是严格的用 final 声明

- 所有的变量和参数都用 final 声明

- 总是在switch语句里加上default

- 用大括号隔开 switch 的每一个 case 块

- []是数组类型的一部分，正：String[] args  反：String args[]

- pojo类中任何布尔类型的变量都不要加is，部分框架解析会引起序列化错误

- pojo类属性必须使用包装数据类型

- 定义pojo类时，不要设定任何属性默认值

- 序列化类新增属性时，不要修改serialversionUID字段，避免反序列化得失败

- pojo类必须写tostring方法，便于抛出异常时，排查问题

- Map/Set的key为自定义对象时，必须重写hashcode和equals

- arraylist的sublist方法得到的结果是原集合的视图，操作会反映到原集合上面

- 使用集合转数组的使用toArray（T[] array）方法，

- Arrays.asList()返回的对象是一个Arrays内部类，并没有实现集合的add，remove等方法，只是转换接口，只有set,get，contain,replace,sort方法，后台的数据仍是原数组

- if else请勿超过三层，若超过三层 则使用状态设计模式

- 循环体内的语句尽量考量性能，定义对象，变量，获取数据库连接等尽量放在外面去

- 后台输送给页面的变量必须加上$!{var} --中间感叹号 如果var=null 或者不存在 那么${var}直接显示在页面上

- 任何数据结构的使用都应该限制大小

- 清理垃圾代码是技术气场

- 对大段代码进行try-catch，这是不负责任的表现，catch时应该分清稳定代码和 不稳定代码，
对于不稳定代码尽量区分异常类型，并做异常处理

- 日志应该依赖使用日志框架SLF4J

- 对trace/debug/info级别的日志输出，必须使用条件输出形式或者使用占位符的方式。

- 日志规约 并发规约  mysql规约

