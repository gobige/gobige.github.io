---
layout: post
title: '使用github pages开启你的独立博客历程'
subtitle: '建立独立博客第一步'
date: 2018-07-9
categories: 技术
author: yates
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: jekyll
---


> 使用github pages开启你的独立博客历程.

### 快速入门
1.在GitHub创建一个新的仓库，并以 username.github.io 的格式命名，username是你的账户名
2.在仓库根目录下创建index.html文件
3.访问https://username.github.io

### 自定义域名访问
如果不喜欢username.github.io的域名访问，想自定义域名访问，那么可以这样做
1.目前github pages支持顶级域名和二级域名访问</p>
2.根目录下创建CNAME文件，文件内容写入域名</p>
3.去dns运营商绑定域名到ip（域名到域名）的解析</p>
 
### 接入jekyll使你的博客更美观
jekyll是一个静态网页生成工具，可以配合github pages使用是你的blog更美观;
[github给出的官方使用jekyll主题文档 →](https://help.github.com/articles/about-jekyll-themes-on-github/)
[jekyll给出的官方使用文档 →](https://jekyllrb.com/docs/home/)

1,请自行百度jekyll主题下载zip，或者找到该主题的git地址,clone该主题，复制该主题的文件到你自己git仓库web项目根目录下面,push到github,访问即可看到效果.（其实这样就很简单的在blog上集成了主题）

因为笔者是在windows平台上搭建的jekyll客户端，jekyll客户端可以在本地进行整个静态文件的build和publish，以及调试，所以建议可以尝试安装jekyll客户端进行静态web的调试，总结下来使用方法如下：
1.[下载ruby+devkit组合套件](https://rubyinstaller.org/downloads/)并在本地进行安装
2.打开cmd命令窗口，输入-gem install jekyll bundler 安装jekyll
3.输入-jekyll -v 验证是否安装成功
4.输入-jekyll new jekyll-website,即可看见初始化新建的一个jekyll模板,这个就是jekyll最基本的一个结构
5.拷贝gemfile文件到你的blog仓库,打开Gemfile文件我们可以看到gem的一个源地址设置（source "https://rubygems.org"），如果访问不通，可以使用修改为[https://gems.ruby-china.org/](https://gems.ruby-china.org/)
6.因为新建模板是自定义minina的，我们可以选择其他模板,比如笔者选择了名为[Cayman Blog]的模板（网上有很多模板，请自行查找），
7.访问[https://rubygems.org gem源地址](https://rubygems.org) 搜索cayman（注意：如果要使用jekyll本地构建须在源地址中找得到该gem才行）
8.拷贝GEMFILE:里面的指令覆盖本地gemfile文件主题项  拷贝安装:里面指令到cmd命令窗口执行 clone gihub的模板文件内容
9.进入待生成的blog目录下，cmd窗口执行 jekyll build

