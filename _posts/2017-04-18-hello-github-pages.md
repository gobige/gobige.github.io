---
layout: post
title: '使用github pages开启你的独立博客历程'
subtitle: '建立独立博客第一步'
date: 2017-07-29
categories: 个人博客
author: yates
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: githubpages
---

很多人都想自建一个博客，在上面写写文章，分享一点东西，但是又不想为此购买服务器，写前端页面，那么有没有一个两全的解决方案呢？答案就是**github pages**

### 快速入门
1. 在GitHub创建一个新的仓库，并以 **username.github.io** 的格式命名，**username**是你的GitHub账户名
2. 在仓库根目录下创建index.html文件
3. 访问https://username.github.io

### 自定义域名访问
如果不想以**username.github.io**的域名访问，想自定义域名访问，那么可以这样做
1. 目前github pages支持**顶级域名**和**二级域名**访问
2. 在新建的blog仓库**根目录**下创建**CNAME**文件，文件内容写入**自定义域名**
3. 去dns运营商**绑定**域名到ip（域名到域名）的解析
 
### 接入**jekyll**使你的博客更美观
**jekyll**是一个静态网页生成工具，可以配合**github pages**使用是你的blog更美观;
[github给出的官方使用jekyll主题文档 ->](https://help.github.com/articles/about-jekyll-themes-on-github/)
[jekyll给出的官方使用文档 →](https://jekyllrb.com/docs/home/)

- 首先请自行百度**jekyll主题**下载zip包，或者找到该主题的git地址,**clone**该主题，复制该主题的文件到你自己git仓库web项目根目录下面,push到github,访问即可看到效果.（其实这样就很简单的在blog上集成了主题）

- 因为笔者是在windows平台上搭建的**jekyll客户端**，jekyll客户端可以在本地进行整个静态文件的**build**和**publish**，以及调试，所以建议可以尝试安装jekyll客户端进行静态web的调试，总结下来使用方法如下：
1. [下载ruby+devkit组合套件](https://rubyinstaller.org/downloads/)并在本地进行安装
2. 打开cmd命令窗口，输入-gem install jekyll bundler 安装jekyll
3. 输入-jekyll -v 验证是否安装成功
4. 输入-jekyll new jekyll-website,即可看见**初始化**新建的一个jekyll模板,这个就是jekyll最基本的一个结构
5. 打开初始化的jeyll模板的根目录，打开名为**Gemfile**的文件，我们可以看到gem的一个源地址设置[https://rubygems.org]（https://rubygems.org），如果访问不通，可以使用修改为[https://gems.ruby-china.org/](https://gems.ruby-china.org/)
6. 因为新建模板自定义模板类型是**minina**，我们可以选择其他模板,比如笔者选择了名为**Cayman Blog**的模板（网上有很多模板，请自行查找），
7. 访问[https://rubygems.org gem源地址](https://rubygems.org) 搜索**cayman**（注意：如果要使用jekyll本地构建须在对应**Gemfile文件**中配置的gem源地址中找得到中才行）
8. 拷贝图中**GEMFILE**:里面的指令覆盖本地**gemfile**文件主题项
9. 修改本地**_config.yml**文件中主题 **theme项** 比如修改为**jekyll-theme-cayman**
10. 拷贝图中**安装**里面指令到**cmd**命令窗口执行
11. clone 图中**gihub**的模板文件内容
9. 进入待生成的blog目录下，cmd窗口执行 **jekyll build**指令，会发现生成了一个**_site**文件夹，该文件就是生成的静态web文件
10. **cmd**窗口执行 **jekyll serve**指令，浏览器访问**localhost:4000**  大功告成！！