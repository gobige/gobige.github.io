---
layout: post
title: '2024最新IDEA插件开发教程'
subtitle: 'IDEA插件开发'
date: 2024-10-13
categories: 分布式
author: yates
cover: 'http://cctv.com'
tags: 插件开发
---

------

## **前言**

参考文档
Jetbrains Idea插件开发文档: https://plugins.jetbrains.com/docs/intellij/welcome.html

开发环境：

idea版本：IntelliJ IDEA 2024.2.1 (Community Edition)
Java版本：jdk21

------

### **前置步骤**

安装**Plugin DevKit插件**

- idea 2023.3 以下版本可以不用安装当前插件。idea自带。2023.3版本及之后的版本没有当前查询，需要自己安装。 
- 插件商店搜索Plugin DevKit用于安装插件


开启**IDEA内部模式**（使用UI检查器定位代码）

- 主菜单栏–>帮助(Help)–>编辑自定义属性
- 输入idea.is.internal=true开启内部模式，重启后生效。

参考链接 https://plugins.jetbrains.com/docs/intellij/enabling-internal.html

**新建插件项目**
- 新建项目-->选择Idea插件(IDE Plugin)

同时注意jdk版本需要和idea对应。参考：https://plugins.jetbrains.com/docs/intellij/creating-plugin-project.html#creating-a-plugin-with-new-project-wizard


**修改Gradle相关配置**


- 修改依赖远程仓库地址，修改build.gradle.kts中的repositories

```java
repositories {
    //    mavenCentral()
    maven {
        url = uri("https://maven.aliyun.com/repository/public")
    }
}
```

- 修改使用本地Idea调试，使用localPath设置本地idea的安装路径（Windows和mac不一样）

```java
intellij {
    localPath.set("D:\\Program Files\\JetBrains\\IntelliJ IDEA 2024.1.2")
    //    version.set("2023.2.6")
    //    type.set("IC") // Target IDE Platform

    plugins.set(listOf(/* Plugin Dependencies */))
}
```
 
**重新加载Gradle变更**

- 设置distributionUrl参数，gradle-wrapper.properties文件配置
```java
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.9-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```
- 点击Gradle图标重新加载变更。（很耗时....）


**防止报错**

- kotlin目录名修改为java。选中kotlin目录，按shift+f6修改目录名为java。不修改后面可能会报错。

------

### **进入开发**
 
**创建Action**

- 使用Plugin DevKit 创建一个Action。
- idea中的action概念可以理解为idea中的任意一个动作，点击了某个按钮，就会执行它绑定的Action类的代码

**plugin.xml配置**
- 创建成功之后就会生成一个Action类和在plugin中注册一个action，表说明他的位置。
 
------

### **测试和运行**

**运行插件**

- 点击gradle运行idea插件，它会重新启动一个idea环境，来运行idea插件。
  


------

## 参考博客：

https://blog.csdn.net/yangfeng20/article/details/139769360
