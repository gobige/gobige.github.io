---
layout: post
title: 'mysql免安装版配置'
subtitle: 'mysql免安装版配置'
date: 2017-09-17
categories: mysql
author: yates
cover: 'http://cctv.com'
tags: mysql
---

1.配置环境变量 PATH,在其后面添加: 你的mysql bin文件夹的路径 (如:C:\Program Files\MySQL\MySQL Server 5.6\bin )

2.自己建立一个my.ini文件,修改一下配置文件\
basedir=C:\Program Files\MySQL\MySQL Server 5.6（mysql所在目录）
datadir=C:\Program Files\MySQL\MySQL Server 5.6\data （mysql所在目录\data）

cmd下执行指令 (双“-” 线)
3.mysqld --initialize 初始化
4.mysqld --install MySQL --defaults-file="D:\dev\mysql-5.7.9-winx64\my.ini"

5.安装好后我们继续执行启动服务的命令：net start MySQL此时会提示服务启动成功

6.安装好后我们继续执行启动服务的命令：net start MySQL此时会提示服务启动成功，因为安装好的MySQL是没有密码的，此时继续执行命令：mysql -u root -p然后回车，
会提示输入密码，因为没有密码我们继续按回车，这里可能会有如下错误信息：mysql ERROR 1045 (28000): Access denied for user

- 解决方法如下：打开my.ini配置文件，在最后一行加上skip-grant-tables然后保存，回到cmd重启Mysql，停止服务命令：net stop MySQL，然后再启动服务：net start MySQL，

7.修改数据库密码update mysql.user set authentication_string=password('root') where user='root' and Host ='localhost';

8.刷新数据库flush privileges

把我们刚才加入的"skip-grant-tables"这行删除，保存退出再重启MySQL就可以了。

六、经过以上步骤我们可以顺利的在cmd中进入数据库了，但我们一般开发的时候为了提高效率都不在cmd里操作数据库，比如我使用的MySQL连接工具为Navicat，我们在使
用此工具连接刚刚配置好的数据库时即使输入了刚刚设置的密码还是会出现连不上的问题，此时我们还需再次用cmd进入数据库设置下密码，即用cmd进入刚刚配置好的数
据库后执行命令：SET PASSWORD = PASSWORD('root');这里我依旧是使用root作为密码，执行完成后再用工具连接即可成功接连。