---
layout: post
title: 'eclipese集成和优化'
subtitle: 'eclipese集成和优化实践'
date: 2017-07-01
categories: idea
author: yates
cover: 'http://cctv.com'
tags: idea
---
 
 ## 1.配置JRE环境
1.1 **JAVA_HOME**
将jdk1.6.rar解压缩到D:\jdk1.6。
右键“我的电脑”的“属性”，选择“高级”，选择“环境变量”，在系统环境里新增（要是存在，就编辑）JAVA_HOME变量(图1.1-1)：
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/1.png) 

图1.1-1
1.2 **CLASSPATH**
添加CLASSPATH变量（图1.2-1），值为**.;%JAVA_HOME%/lib/dt.jar;%JAVA_HOME%/lib/tools.jar;**

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/2.png)
图 1.2-1

1.3 **Path**
在“Path”里添加JAVA_HOME的bin目录（图1.3-1）：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/3.png)
图 1.3-1

点击确定后重启CMD命令窗口。
在命令窗口中输入：java –version，显示结果如下(图1.3-2)就表示配置正确：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/4.png)
图 1.3-2

## 2.配置Maven环境
2.1 **M2_HOME**
将maven3.rar解压缩到D:\maven3。
右键“我的电脑”的“属性”，选择“高级”，选择“环境变量”，在系统环境里新增（要是存在，就编辑）M2_HOME变量(图2.1-1)：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/5.png)
图 2.2-1

点击确定。
2.2 **Path**
在Path里添加bin目录的路径(图2.2-1)：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/6.png)
图 2.2-1

点击确定后，重启CMD命令窗口，查看maven版本：mvn –v，显示如下(图2.2-2)就表示配置正确。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/7.png)
图 2.2-2

## 3. 配置IDE环境

3.1 **安装eclipse**
解压eclipse.rar到D:\eclipse。

双击eclipse.exe启动IDE。

选择eclipse的“window”->“Prefernces”，输入workspace（或是手动找也可以）：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/8.png)
图 3.1-1
将GBK改为UTF-8即可(图3.1-1)。
3.2 **配置Maven环境**
选择eclipse的“window”->“Prefernces”->“Maven”。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/9.png)
图 3.2-1

修改Installations：点击“Add...”（图3.2-1），选择D:\maven3\apache-maven-3.0.3（图3.2-2）。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/10.png)
图 3.2-2

修改User Settings(图3.2-3)：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/11.png)
图 3.2-3

点击“Browse...”，文件路径为：D:\maven3\settings.xml(图3.2-4)。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/12.png)
图3.2-4

3.3 **eclipse集成Tomcat**
将Tomcat7.0.12_8079.rar解压缩到D:\Tomcat7.0.12_8079。
右键eclipse左边的工作区，在弹出的窗口中选择“New”->“Other...”，在窗口中选择“server”，点击“Next”(图3.3-1)。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/13.png)
图3.3-1

在窗口中选择Tomcat v7.0 server（要是用的tomcat6.0.x就选择v6.0），点击“Next”(图3.3-2)。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/14.png)
图 3.3-2

将jre改为jdk1.6，然后点击“Browse...”(图3.3-3)，

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/15.png)
图 3.3-3

选择tomcat的安装路径：D:\Tomcat7.0.12_8079。
点击“Finish”(图3.3-4)。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/16.png)
图3.3-4

看到下面这张图代表集成Tomcat成功(图3.3-5)。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/17.png)
图 3.3-5
接下需要修改Tomcat的部分参数(图3.3-6)：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/18.png)
图 3.3-6

将“Server Locations”选择第二个；将“Deploy path”改为webapps；将“Timeouts”的Start时间改为450000、将Stop的时间改为150，然后按Ctrl+s保存即可（图3.3-7）。如下图所示：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/19.png)

## 4.eclipse工作空间优化
1.Eclipse安装目录下：D:\eclipse，打开eclipse.ini：
将下列参数改为：
-vmargs
-Xmx768m
-XX:MaxPermSize=512M
-XX:ReservedCodeCacheSize=128m
-Xmx768m 可以先试着改成1024，若无法启动稍微改小点。

2、IDE里的windows->preferences->validation：把验证全部disable：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/1_2.png)

3、IDE里的windows->preferences->startup：把启动项全部取消，除了tomcat 6：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/2_2.png)

4、IDE里的windows->preferences->maven里的download respository…取消。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/3_2.png)

修改编码方式：
修改workspace编码方式

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/4_2.png)

修改JSP编码方式：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/5_2.png)

修改HTML编码方式：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/6_2.png)

5、启动服务器时报无法找到class的错误
造成这个问题的根由是IDE由于不稳定在并没有成功的将.java文件编译成.class文件，解决办法如下
a、Project->关掉Build Automatically。
b、按照项目的引用顺序从b2b_interface开始，先执行Run as Maven clean,再执行Project下面的clean，执行完后查看本地视图下面指定编译目录下面是否有编译成的class文件，如果没有则重复执行Run as Maven clean和Project的clean，直到编译成功为止。
c、b2b_service和b2b_web也按照这个步骤执行，编译好后把Build Automatically开启。


![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/eclipse/20.png)

打开Eclipse，菜单栏，点击Window，点击preferences；
2.在左侧选择General-Editors-Test Editors;
3..在右侧下方,选择Background color,并点击取消右边System Default选择勾;  (84   91  205)