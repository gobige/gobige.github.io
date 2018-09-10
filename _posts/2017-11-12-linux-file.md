---
layout: post
title: 'linux学习之liunx文件'
subtitle: 'linux文件，目录结构，权限相关知识'
date: 2017-11-12
categories: linux
author: yates
cover: 'http://cctv.com'
tags: linux
---

### liunx目录结构
- /bin:存放经常使用的命令
- /boot Linux启动核心文件，链接文件，镜像文件
- /dev 存放linux外部设备
- /etc 系统管理所需配置文件和子目录
- /home 用户目录
- /lib 系统共享目录
- /lost+found 系统非正常关机后存放的一些文件
- /media系统挂载设备目录
- /mnt临时挂载别的文件系统的
- /opt系统额外安装软件目录
- /proc系统内存映射目录
- /root超管使用主目录
- /sbin系统管理员使用系统管理程序
- /selinux redhat,centos特有安全目录
- /srv服务启动后需要提取的数据
- /usr 应用程序和文件存放目录
- /tmp 存放一些临时文件
- /usr/bin 系统用户使用应用程序
- /usr/sbin 超级用户使用比较高级的管理程序和系统守护程序
- /usr/src 内核源代码默认的放置目录
- /var 经常修改的文件存放于此，日志文件


### linux文件目录权限

我们可以通过ls -l来查看当前目录下的文件和目录权限情况

![请输入图片地址](http://muyibeyond.cn/img/2017-11-12-linux-relation/1.png)

linux文件权限以分组划分，组下面有用户，用户和组对应关系是多对多

从左到右一串字母分别代表
- 0：文件类型 d（目录文件）
- 1-3：属主权限 分别对应rwx，读，写，执行
- 4-6：属组权限 分别对应rwx，读，写，执行
- 7-9：其他用户权限 分别对应rwx，读，写，执行  

**更改文件权限**

chgrp 属组名 文件名  -更改文件属组
chown 属主名 文件名  -更改文件属主
chmod xyz 文件或目录 -更改文件权限 xyz分别代表owner，group，ohters的权限，以数字赋值，权限rwx对应数字权限分别是4,2,1 比如 chmod 777 testfile

**常用文件操作(增删改)指令**

- ls 列出目录
    - ls -a 所有文件
    - ls -d 只列出文件夹
- cd 切换目录
- pwd 显示当前路径
- mkdir 创建一个新文件夹
    - mkdir test 创建test文件夹
    - mkdir -m 777 test 创建权限为777的test文件夹
- rmdir 删除一个**空**的目录
    - rmdir test 删除test文件夹
    - rmdir -p test/test2/test3 删除多个文件夹
- cp 拷贝文件或目录
    - cp file1 ../file2
- rm 移除文件或目录
    - rm -f test 强制删除test文件或目录
    - rm -f -r 递归删除文件目录
- mv 一点目录或文件
    - mv -f 强制移动文件和目录
    - mv -u 目标存在且比较新，才会升级

**查看文件**

- cat 查看文件所有内容
    - cat test 查看test文件内容
    - cat -n test 显示行号
    - cat -v 列出特俗字符 
- tac cat指令的倒装
- nl 显示带行号的文件内容
- more 可一页一页翻动文本内容（空白键 下翻一页；enter 下翻一行；/str 查找str关键字；b 向上翻页）
- less 一页一页翻动文本内容（pagedown/空白键 向下翻页；pageup向上翻页；/str 向下查找str关键字；?str向上搜子串；n 重复前一个搜索）
- head 取出文件前几行
    - head -n 20 取出前20行
- tail head指令倒装
    - tail -f 取出文件后多少行

**文件对比，识别，查找，创建，统计**

- cmp [file1] [file2] 对比两个文件是否相同，返回第一个不相同的位置
- diff [file1] [file2] < 标识后面文件比前面文件少的内容，> 表示前面文件比后面文件多的内容
- file [file1] 显示文件类型
- find 查找文件
    - find . -name "*.txt" 在当前目录下及子目录下查询后缀为txt的文件
    - find . -ctime -20 查找当前目录及子目录 20天内更新过文件列出
- touch 修改文件或目录时间属性，若不存在则创建文件
    - touch [file] 创建file文件
- which 在环境变量$PATH设置的目录里查找符合条件的文件
- whereis 查找二进制，源代码，man手册页，
- wc 统计文件行数，字数，字节数




**磁盘相关**

- df 列出文件系统整体磁盘使用量
    - df -h 显示磁盘容量以较为容易方式展示

yum check-update 查看可更新软件列表
yum install <package> 安装指定软件命令
yum update <package> 更新指定软件
yum remove <package> 删除软件包命令


**网络相关**

- ifconfig 显示或设置本机网络设备
- netstat -i 显示万科列表
- telnet 远程登录主机
- ping 检查主机连通性


**系统相关**

- kill [pid] 杀死进程
- kill -KILL [pid] 强制杀死进程
- kill -s 9 [pid] 彻底杀死进程
- last 显示系统开机以来每月初登入者信息
    - last -3 展示最近3个 用户
- date 显示系统时间
- lastb 显示登录失败用户信息
- logname 显示当前登录用户名称
- ps 显示进程信息
    - ps -u root 显示root用户信息
- showdown 关闭linux服务器
    - showdown -h now 立即关闭linux服务器
- top 实时显示系统状态
- pstree 以树状图显示所有行程状态
- reboot 重启linux服务器
- sudo 以管理员身份执行
- su 更换使用者身份
- free 显示内存状况
- clear 清屏
- export 设置显示环境变量
    - export -p 显示当前环境变量
    - export hostname=yates 设置环境变量
- passwd 修改用户密码
- rdate 显示远端主机日期和时间
    - rdate -p 192.168.1.120 显示远端主机日期密码

- gzip 压缩文件
    - gzip [file1][file2] 压缩文件1，文件2
- unzip 解压文件
- dump 备份文件
- restore 还原备份文件