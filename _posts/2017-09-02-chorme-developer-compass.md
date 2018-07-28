---
layout: post
title: 'Chrome 浏览器开发者工具教程'
subtitle: 'Chrome 浏览器开发者工具教程'
date: 2017-09-02
categories: 开发工具
author: yates
cover: ''
tags: devTool
---
### 前言
随着可用于提高生产力和简单调试的功能和功能，Chrome开发工具已经成为任何前端开发人员或网站安全研究员必须知道的软件。虽然简单，有效的用户界面和用户体验，它为任何操作、分析、内存诊断和 JavaScript 调试提供了一个单一的进程。本文旨在对ch'rome开发工具不同功能进行综述，大部分内容可外推Firefox和edge等浏览器

## 进入开发者模式

* 鼠标右键点击查看元素，选择inspect(检查)
* 快捷键 ‘Ctrl + Shift + I’
* 浏览器右上角 点击Customize chrome -> 更多工具 -> 开发者工具 

## [Elements]自由编辑你的网站布局
进入开发者模式后，我们会发现整个网页的dom元素结构和html，css内容展示了出来，当鼠标移到相应代码上会聚焦相应的代码区域到网页

![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/1.gif)

## [Device Toolbar]设备工具栏-检查网站的响应
为了建立响应式网站，Chrome 开发工具有"Toggle 设备工具栏"(左上角)选项，可以用来以不同的分辨率查看网站。 它还为 iPhone、 iPad 和 Nexus 等移动设备提供了特定的视图

![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/2.png)

## [Sources] 调试 JS
最常用的 Chrome 开发工具面板，可用于分析、调试和编辑网站的 JavaScript，HTML 和 CSS

格式化代码：
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/3.png)

打断点：
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/4.png)
可以通过在小窗口中打开 * 控制台 * * 面板来调查各个变量的值。

## [Audits]最佳测试实践
这个面板可以用来识别影响网站性能、可访问性和用户体验的常见问题和问题。

![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/5.png)
使用谷歌的灯塔项目作为后台。 通常的检查包括检查 Web 应用程序的标准、性能指标、最佳实践和可访问性。 只需要到面板上，运行一个有期望的选项的审计，以获得结果

## [Network]优化页面负载性能和调试请求问题
此面板可用于监测网站提出的任何类型的请求，
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/6.png)

- Cookies 检查与请求一起发送的cookie和设置响应的cookie
- Timing 时间，显示时间的要求统计数据，可用于性能诊断
- presets 设置网络环境 在3g/4g或者慢 离线工作
右键点击一个网络请求的兴趣，并选择一个选项来复制请求。 最常用的选项是复制 cURL，这样可以获得 cURL 等效的请求并在控制台上重发。

## [Application]审查你的网站资源
这个面板用于管理网站加载的各种资源。这包括cookies、本地存储、应用程序缓存、图像、字体、样式表、注册服务工作者、会话存储、 Web SQL 数据库和 IndexedDB
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/7.png)

这些资源的值可以通过这个面板来检查、设置或清除。 它还显示了所有这些数据单独和联合使用了多少存储空间
- manifest 检查和触发 Web 应用程序的清单
- Service Workers 用来执行与服务工作者相关的任务，如未注册、停止、离线等
- Clear Storage 清除所有的存储、缓存和服务工作者
- Frames 通过各种过滤器来组织资源
- storage 查看和编辑各种存储和数据

## [Memory]追踪内存泄漏
使用这个面板，我们可以在大多数常见的场景中找到影响页面性能的问题，包括内存泄漏和错误
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/8.png)

- Heap Snapshot 用来拍摄**堆**的快照，它显示了在网站的 JavaScript 对象和 DOM 节点之间的内存分布。 这可以用来找到可能导致内存泄漏的分离的DOM树(搜索"分离")。红色中突出的节点在代码中没有直接引用，而黄色节点则有直接引用。 红色的树之所以活着是因为它们是黄节点树的一部分。特别是，我们需要关注黄色节点。 确保黄色节点的存活时间不超过需要的时间
- Record Allocation Timeline记录分配时间表:这个记录有助于跟踪网站JS堆中的内存泄漏。开始记录，执行你怀疑的记忆泄漏的动作，然后按停止记录。记录中的蓝条表示新的内存分配。这些是内存泄漏的可能候选项，可以放大以筛选构造函数窗格，然后可以在对象窗格中查看特定对象
- Record Allocation Profiles分配配置文件:这是从你的JavaScript函数中显示的内存分配。类似于时间轴，你可以用你想要调查的动作来记录档案。开发工具将显示每个函数的内存分配。然后你可以调查那些看起来是记忆力大的罪魁祸首的功能

## [Performance] 提高页面的运行时性
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/9.gif)

这个面板允许我们分析应用程序的JavaScript的运行时性能。该面板记录了在页面生命周期内发生的各种事件，并显示每个页面所花费的时间。一旦选择截屏选框，我们可以使用网络和CPU的节流选项从捕获设置来模拟我们的网站在移动设备上的性能。 使用记录按钮开始记录将开始分析网站的各种生活事件。几秒钟后，点击Stop将显示每个事件的结果和时间。 分析每个图表，以了解网站的哪些部分是有辱人格的表现。在每张图表上方盘旋，将显示当时页面的截图。 这些录音也可以通过使用鼠标重放。

## [Security] 检查常见的安全问题
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/10.png)

这个相对较新的面板允许您测试您的网站最常见的安全措施。 对不同资源的所有来源都进行了有效的 SSL 证书、安全连接、安全资源和其他事项。 起源还根据不安全 / 安全原则进一步过滤到不同类别，以便于对问题进行跟踪。 只需打开开发工具，选择这个面板并重新加载你的网站以获得一个分析。 在未来，更多的安全工具可能被添加到这个面板中。

## [Console] 日志诊断信息
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-09-02-chorme-developer-compass/11.png)

控制台面板可以被认为是一个实验操场。 您可以运行任何类型的 JavaScript 代码，可以访问全局窗口变量。 控制台面板还可以作为各种错误(网络和源代码相关)的记录器，并在发生错误的地方将所有错误显示在一个网站或源代码中的行号。 此外，在调试过程中，控制台面板可以打开，以检查当前断点处的局部变量值。 控制台面板公开了一个对象'控制台'，可以用来以不同格式记录信息。


[谷歌的Chrome DevTools官方详细教程](: https://developers.Google.com/web/tools/Chrome-DevTools)