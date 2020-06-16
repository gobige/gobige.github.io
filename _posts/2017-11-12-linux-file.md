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

![请输入图片地址](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/1.png)

linux文件权限以分组划分，组下面有用户，用户和组对应关系是多对多

**第一个字段** 从左到右一串字母分别代表

- 0：文件类型 
    - d：目录文件
    - -：普通文件
    - l：指令
- 1-3：属主权限 分别对应rwx，读，写，执行
- 4-6：属组权限 分别对应rwx，读，写，执行
- 7-9：其他用户权限分别对应rwx，读，写，执行  

**第二个字段** 硬链接数目
**第三个字段** 所属用户
**第四个字段** 所属组
**第五个字段** 文件大小
**第六个字段** 文件修改日期
**第五个字段** 文件名

**更改文件权限**

- 更改文件属主 chown 属主名 文件名  
- 更改文件属组 chgrp 属组名 文件名  
- 更改文件权限 chmod xyz 文件或目录

 xyz：代表owner，group，ohters的权限，
 数字赋权：权限rwx对应数字权限分别是4,2,1  例： chmod 777 testfile

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

- file [file1] 显示文件类型
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

**文件编辑**

- vim <file>：vim文本编辑器
    - i：insert
    - esc：退出编辑
    - :w ：保存
    - :wq：保存退出
    - q!：强制退出

**文件对比，识别，查找，创建，统计**

- 文件对比
	- cmp [file1] [file2] 对比两个文件是否相同，返回第一个不相同的位置
	- diff [file1] [file2] < 标识后面文件比前面文件少的内容，> 表示前面文件比后面文件多的内容
- find 查找文件
    - find . -name "*.txt" 在当前目录下及子目录下查询后缀为txt的文件
    - find . -ctime -20 查找当前目录及子目录 20天内更新过文件列出
- touch 修改文件或目录时间属性，若不存在则创建文件
    - touch [file] 创建file文件
- 文件查找
	- which 在环境变量$PATH设置的目录里查找符合条件的文件
	- whereis 查找二进制，源代码，man手册页，
	- grep xx.txt aa.txt 查找文件
- 文件统计
	- wc 统计文件行数，字数，字节数

**文件备份，恢复**

- gzip 压缩文件
    - gzip [file1][file2] 压缩文件1，文件2
- unzip 解压文件
- dump 备份文件
- restore 还原备份文件

**程序运行**

- ./filename：运行程序
- nohup：后台挂起运行程序
    - nohup comand > out.file 2>&1 &（1标准输出2标准错误输出 合并到out.file）
- systemctl：以服务运行程序

**磁盘相关**

- df 列出文件系统整体磁盘使用量
    - df -h 显示磁盘容量以较为容易方式展示

**软件包安装**

wget <download.xxxx.cn>：下载工具

下载的tar，zip包要配置path路径才能使用

**CentOS**
安装rpm安装包：rpm -i xxx.rpm
rpm -qa |grep <name> 查看是否存在安装
rpm -e：删除软件包
yum check-update 查看可更新软件列表
yum list <name> 检索包含name的列表
yum install <package> 安装指定软件命令
yum update <package> 更新指定软件
yum remove <package> 删除软件包命令

下载服务端配置文件：/etc/yum.repos.d/CentOS-Base.repo

**Ubuntu**
deb安装包：dpkg -i xxx.deb
dpkg -l：查看安装软件列表
dpkg-r：删除软件包
apt-get install <package>安装指定软件命令

下载服务端配置文件：/etc/apt/sources.list



**网络相关**

- ifconfig 显示或设置本机网络设备
- netstat -i 显示网卡列表
- telnet 远程登录主机
- ping 检查主机连通性

**防火墙设置**

停止防火墙
systemctl stop firewalld

锁定防火墙
systemctl mask firewalld

安装iptables-services：
yum install iptables-services

设置开机启动
systemctl enable iptables

重启
systemctl restart iptables

/etc/sysconfig/iptables添加端口开放设置
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

保存设置
service iptables save


**系统相关**

- passwd ：修改密码
    - passwd yates：修改指定用户密码
- useradd：新建用户，在etc/passwd文件存储


**系统信息查看**

- ps 显示进程信息
    - ps -u root 显示root用户信息
- top 实时显示系统状态
	- 系统统计信息：当前时间；系统运行实际；当前用户数；平均负载（1m,5m,15m）
	- 进程，cpu统计信息：
		- VIRT：进程虚拟内存大小，包含没有真正物理分配
		- RES：常驻内存大小，进程实际使用物理内存大小，不包括Swap和共享内存
		- SHR：共享内存大小，与其他进程共享内存，加载动态链接库及程序代码段
		- %MEM：进程使用物理内存占系统总内存百分比
	- 内存信息：swap used 超过0基本确定内存瓶颈问题
- pstree 以树状图显示所有行程状态
- free 显示内存状况
- export 设置显示环境变量
    - export -p 显示当前环境变量
    - export hostname=yates 设置环境变量
- vmstat 监控进程上下文切换情况
	- procs：
		- r：等待运行的进程数
		- b：处于非中断睡眠状态的进程数
	- memory：
		- swpd：虚拟内存使用情况
		- free：空闲的内存
		- buff：用来作为缓冲的内存数
		- cache：缓存大小
	- swap：
	    - si：从磁盘交换到内存的交换页数量
	    - so：从内存交换到磁盘的交换页数量
	- io：
	    - sibi：发送到块设备的块数
	    - bo：从块设备接收到的块数
    - system：
        - in：每秒中断数
        - cs：每秒上下文切换次数
    - cpu
        - us：用户CPU使用时间
        - sy：内核CPU系统使用时间
        - id：空闲时间
        - wa：等待I/O时间
        - st：运行虚拟机窃取的时间
- pidstat：监测具体线程上下文切换
    - u：默认参数，显示各个进程的cpu使用情况；
    - r：显示各个进程的内存使用情况；
    - d：显示各个进程的I/O使用情况；
    - w：显示每个进程的上下文切换情况；
    - p：指定进程号；
    - t：显示进程中线程的统计信息
    - cswch/s：每秒主动任务上下文切换数量
    - nvcswch/s：每秒被动任务上下文切换数量
- iostat：提供了每个磁盘的使用率、IOPS、吞吐量等各种常见的性能指标

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/10.png)

**用户信息查看**

- last 显示系统开机以来每月初登入者信息
    - last -3 展示最近3个 用户
- lastb 显示登录失败用户信息
- date 显示系统时间
- logname 显示当前登录用户名称
- sudo 以管理员身份执行
- su 更换使用者身份
- rdate 显示远端主机日期和时间
    - rdate -p 192.168.1.120 显示远端主机日期密码

**小心使用**

- kill [pid] 杀死进程
- kill -KILL [pid] 强制杀死进程
- kill -s 9 [pid] 彻底杀死进程 // 9无条件强制杀死 15 尽量杀死，调用钩子函数
- showdown 关闭linux服务器
    - showdown -h now 立即关闭linux服务器
- reboot 重启linux服务器
- clear 清屏
- passwd 修改用户密码


**linux插件**

lrzsz是一款在linux里可代替ftp上传和下载的程序：  yum install lrzsz  -y  

**配置相关**

- echo $PATH  显示系统环境变量配置
- export PATH=$PATH:/usr/{name} 临时添加添加新的系统环境变量。etc/profil文件添加才能永久添加，然后使用source指令生效配置)


**进程**

创建进程的系统调用叫fork，父进程创建子进程。对应fork调用返回值，如果当前进程是子进程，返回0，父进程返回子进程号。通过if语句判断，若是子进程调用**execve系统调用**来执行另一个程序。父进程通过**waitpid系统调用**，查看子进程运行情况

**进程内存空间**

- **数据段**：进程运行中产生数据的部分
- **堆**：动态分配的变量，长期保存

brk和mmap 分配堆内内存

程序运行起来，会在**/proc**下面有对应进程号，一系列文件

**sigaction系统调用**，注册一个信号处理函数

msgget创建一个新队列，**msgsnd**将消息发送到消息队列，接收方使用**msgrcv**从队列中取消息；通信信息比较大时，使用**shmget**创建共享内存块方式。通过**shmat**将共享内存映射到自己内存空间。当共享内存产生竞争时，通过**sem_wait**获取信号量，**sem_post**释放信号

Glibc：提供丰富的API，除了字符串处理，数学运算等用户态服务，封装了操作系统提供系统服务。

**线程的数据**：

- 线程栈上本地数据：局部变量；ulimit -a查看栈大小，ulimit -s修改栈配置
- 整个进程全局变量：共享变量
- 线程私有数据：pthread_key_create函数创建

Mutex锁：pthread_mutex_init初始化mutex锁，pthread_mutex_lock获取锁，pthread_mutex_unlock释放锁

**进程数据结构**

Linux内核通过一个链表将所有task_struct串起来

### **task_struct**

**任务ID**
```java
pid_t pid; // process id
pid_t tgid;// thread group ID
struct task_struct *group_leader; 
```

**信号处理**

定义哪些信号阻塞不处理，哪些信号等待处理，哪些通过信号处理函数进行处理

**任务状态**

```java
volatile long state;    // -1 unrunnable, 0 runnable, >0 stopped
int exit_state;
unsigned int flags;
```

- 可中断睡眠状态：浅睡眠状态，等待信号到来，处理信号。例如kill信号
- 不可中断睡眠状态：深度睡眠，不可被信号唤醒，死等I/O操作完成。
- 可以终止的新睡眠状态：可响应致命信号，如：kill

**进程调度**

调度实体，优先级，调度器类，调度策略，可使用CPU，是否运行队列

**实时进程比普通进程优先级高**

**调度实体**

```java
struct sched_entity se;
struct sched_rt_entity rt;
struct sched_dl_entity dl;
```

进程根据自己是实时的，还是普通的类型，通过调度实体变量将自己挂载某数据结构里。如：普通进程，通过sched_entity(包含了vruntime和权重load_weight等参数)挂载红黑树。

每个CPU都有自己的 **struct rq** 结构，包括一个实时进程队列**rt_rq**和一个CFS运行队列**cfs_rq，CFS的队列是一棵红黑树，树的每一个节点都是一个sched_entity，每个sched_entity都属于一个task_struct，task_struct里面有指针指向这个进程属于哪个调度类


主动调度的过程，也即一个运行中的进程主动调用__schedule让出CPU。在__schedule里面会做两件事情，第一是选取下一个进程，第二是进行上下文切换。而上下文切换又分用户态进程空间的切换和内核态的切换。

**抢占式调度和主动式调度**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/26.png)


**运行统计信息**

utime,stime,nvcsw,nivcsw,start_time,real_start_time

**进程亲缘关系**

parent，children，sibling（当前进程插入到兄弟链表中）

**进程权限**

Objective和Subjective：谁能操纵我，我能操纵谁

**内存管理**

```java
struct mm_struct   *mm;
struct mm_struct   *active_mm;
```

**文件与文件系统**
```java
/* Filesystem information: */
struct fs_struct                *fs;
/* Open file information: */
struct files_struct             *files;
```

**内核栈和进程运行紧密相关。**

- 用户态，应用程序至少一次函数调用，32位系统传递参数使用函数栈，64位前6参数用寄存器，其他用函数栈
- 内核态，32位和64位都使用内核栈。前者使用thread_info 和 task_struct 进行关联，后者使用 Per-CPU变量变量

## **fork创建进程，**

**先copy_process，复制结构。**

- 调用alloc_task_struct_node分配task_struct结构
- 调用alloc_thread_stack_node来创建内核栈
- 调用arch_dup_task_struct的，将task_struct进行复制
- 调用setup_thread_stack设置thread_info

copy_creds权限赋值

- 调用prepare_creds，从内存中分配一个新的struct cred结构，然后调用memcpy复制一份父进程的cred
- 将新进程的“我能操作谁”和“谁能操作我”两个权限都指向新的cred

copy_process重新设置进程运行的统计量

然后copy_process开始设置调度相关的变量

- 设置进程的实际运行时间和虚拟运行时间等为0
- 设置进程的状态p
- 初始化优先级
- 设置调度类
- 调用调度类的task_fork函数

**唤醒新进程**

- 将进程的状态设置为TASK_RUNNING

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-11-12-linux-relation/27.png)

## 线程的创建

**用户态创建线程**

Glibc库的一个函数 pthread_create进程线程创建

- 设置的线程栈大小。如果没有传入线程属性，就取默认值
- pthread结构维护线程的结构
- ALLOCATE_STACK，创建线程栈
	- 设置线程属性栈的大小
	- 末尾有一块空间guardsize为了防止栈的访问越界，一旦访问到这里就报错
	- 线程栈是在进程的堆里面创建，get_cached_stack作为线程栈不断 申请和清除 使用的内存缓存块
	- 如果没有缓存，调用__mmap创建
	- 线程栈自顶向下生长
	- 计算出guard内存位置，调用setup_stack_prot设置这块受保护内存
	- 填充pthread这个结构里面的成员变量
	- 将线程栈放到stack_used链表，一旦线程结束，先缓存起来，放到stack_cache链表，等其他线程创建使用

**内核态创建任务**

## 内存管理

- 虚拟内存空间的管理，每个进程看到的是独立的、互不干扰的虚拟地址空间
- 物理内存的管理，物理内存地址只有内存管理模块能够使用
- 内存映射，需要将虚拟内存和物理内存映射、关联起来

**分段机制、分页机制以及从虚拟地址到物理地址的映射方式**

- 虚拟内存空间的管理，将虚拟内存分成大小相等的页
- 物理内存的管理，将物理内存分成大小相等的页
- 内存映射，将虚拟内存也和物理内存也映射起来，并且在内存紧张的时候可以换出到硬盘中

**物理内存组织方式**

- 平坦内存模型：物理地址连续，页也是连续，每个页大小一样。任何一个地址，直接除一下每页的大小，容易直接算出在哪一页（多个CPU访问内存经过总线**SMP**，总线称为瓶颈）
- 非连续内存模型 NUMA：每个CPU有自己本地内存，CPU访问本地内存不用过总线，速度快很多,称为NUMA节点。本地内存不足时，向另外的NUMA节点申请内存，访问延时比较长

DMA：CPU只需向DMA控制器下达指令，让DMA控制器来处理外设和内存之间数据的传送，数据传送完毕再把信息反馈给CPU

- 如有多个CPU，就有多个节点。每个节点用struct pglist_data表示，放在一个数组里面。
- 每个节点分为多个区域，每个区域用struct zone表示，也放在一个数组里面。
- 每个区域分为多个页。为了方便分配，空闲页放在struct free_area里面，使用伙伴系统进行管理和分配，每一页用struct page表示。
- kswapd负责物理页面的换入换出；
- Slub Allocator将从伙伴系统申请的大内存块切成小块，分配给其他系统

