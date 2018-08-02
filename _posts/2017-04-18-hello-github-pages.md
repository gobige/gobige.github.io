---
layout: post
title: '使用github pages开启你的独立博客历程'
subtitle: '建立独立博客第一步'
date: 2017-04-18
categories: 个人博客
author: yates
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: githubpages
---


### 前言
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
[github给出的官方使用jekyll主题文档->](https://help.github.com/articles/about-jekyll-themes-on-github/)

[jekyll给出的官方使用文档 →](https://jekyllrb.com/docs/home/)

- 首先请自行百度**jekyll主题**下载zip包，或者找到该主题的git地址,**clone**该主题，复制该主题的文件到你自己git仓库web项目根目录下面,push到github,访问即可看到效果.（其实这样就很简单的在blog上集成了主题）

* 因为笔者是在windows平台上搭建的jekyll客户端，jekyll客户端可以在本地进行整个静态文件的build和publish，以及调试，所以建议可以尝试安装jekyll客户端进行静态web的调试，总结下来使用方法如下：
1. [下载ruby+devkit组合套件](https://rubyinstaller.org/downloads/)并在本地进行安装
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-04-18-hello-github-pages/1.png)

2. 打开cmd命令窗口，输入-gem install jekyll bundler 安装jekyll
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-04-18-hello-github-pages/2.png)

3. 输入-jekyll -v 验证是否安装成功

4. 输入-jekyll new jekyll-website,即可看见初始化新建的一个jekyll模板,这个就是jekyll最基本的一个结构

5. 打开Gemfile文件我们可以看到gem的一个源地址设置（source "https://rubygems.org"），如果访问不通，可以使用修改为[https://gems.ruby-china.org/](https://gems.ruby-china.org/)

6. 因为新建模板是自定义minina的，我们可以选择其他模板,比如笔者选择了名为[Cayman Blog]的模板（网上有很多模板，请自行查找），

7. 访问[https://rubygems.org gem源地址](https://rubygems.org) 搜索cayman（注意：如果要使用jekyll本地构建须在源地址中找得到该gem才行）

8. 拷贝GEMFILE:里面的指令覆盖本地gemfile文件主题项 修改_config.yml文件中主题 theme: jekyll-theme-cayman   拷贝安装:里面指令到cmd命令窗口执行 clone gihub的模板文件内容
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-04-18-hello-github-pages/3.png)
![此处输入图片的描述](http://www.muyibeyond.cn/img/2017-04-18-hello-github-pages/4.png)

9. 进入待生成的blog目录下，cmd窗口执行 jekyll build，会发现生成了一个_site文件夹

10. cmd窗口执行 jekyll serve指令，浏览器访问localhost:4000  大功告成！！


#### 一个基本的jekyll的静态网站目录结构如下

>* _config.yml
>* _drafts
>>* ├── begin-with-the-crazy-ideas.textile
>>* └── on-simplicity-in-technology.markdown

>* _includes
>>* ├── footer.html
>>* └── header.html

>* _layouts
>>* ├── default.html
>>* └── post.html

>* _posts
>>* ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
>>* └── 2009-04-26-barcamp-boston-4-roundup.textile

>* _data
>>* └── members.yml

>* _site
>* index.html

每个目录解释如下
**_config.yml**
保存配置数据。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。

**_drafts**
drafts 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。学习如何使用 drafts.

**_layouts**
layouts 是包裹在文章外部的模板。布局可以在YAML头信息中根据不同文章进行选择。这将在下一个部分进行介绍。标签  {{ content }} 可以将content插入页面中。

**_posts**
这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。 The permalinks 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。

**_site**
一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中。
index.html and other HTML,Markdown,Textilefiles如果这些文件中包含YAML头信息部分，Jekyll就会自动将它们进行转换。当然，其他的如.html，.markdown，.md，或者.textile等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。

**Other Files/Folders**
其他一些未被提及的目录和文件如  css 还有 images 文件夹， favicon.ico 等文件都将被完全拷贝到生成的 site 中。