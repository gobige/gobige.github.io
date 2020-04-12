---
layout: post
title: 'Kubernetes'
subtitle: 'Kubernetes学习'
date: 2019-02-29
categories: 容器
author: yates
cover: 'www.baidu.com'
tags: 容器
---

# 前言

容器技术的兴起起源于PaaS技术的普及，Docker项目通过容器镜像，解决了应用打包这个根本性难题，容器本身没有价值，有价值的是容器编排

## 什么是容器

容器是一种沙盒技术，把你的应用装起来，应用与应用之间，因为有了边界而不至于相互干扰，而也方便就随机移动。

## 虚拟机与容器不同

虚拟机通过硬件虚拟化功能，模拟出运行一个操作系统需要的硬件，然后在虚拟硬件上安装一个新的操作系统。会带来额外的资源消耗
，实验证明，运行CentOS的KVM虚拟机启动后，不做优化情况下，虚拟机自己需要占用100~200MB内存。

容器化后的用户应用，本质还是一个宿主机上普通进程。所以多了容器之间使用的还是一个操作系统内核。其次Linux内核中，很多资源和对象时不能被Namespace化的，例如：时间

## 容器技术实现

- Cgroups技术用来制造约束主要手段：限制一个进程组能够使用资源上限，CPU,内存，磁盘，网络等；对进程进行优先级设置
	- Linux Cgroups：一个子系统下面，为每个容器创建一个控制组，启动容器进程后，把进程PID填写到对应控制组的tasks文件中
	
- Namespace技术用来修改进程视图；使新创建的进程只能看到重新计算过的进程编号
	- Mount Namespace：让隔离进程只看到Namespace的挂载点消息。（对容器进程视图改变，一定伴随着挂载操作才能生效）
	- Network Namespace：让隔离进程只看到当前Namespace的网络设备和配置

使用Linux中的chroot命令在容器进程启动前挂载整个根目录"/"，改变进程根目录到你指定的位置。为了让容器根目录更真实，一般会在容器根目录下挂载一个完整操作系统的文件系统。称为rootfs，俗称容器镜像。在 /var/lib/docker/aufs/mnt 目录下

为了解决每开发一个应用都需重复制作一次rootfs的问题，Docker在镜像的每一步操作都生成一个层，一个增量rootfs。而多个增量rootfs联合挂载一个完整rootfs方案。

## docker exec指令

Linux Namespace创建的隔离空间虽然看不见摸不着，但一个进程Namespace信息在宿主机以文件方式存在。对应在/proc/[进程号]/ns 下有一个对应虚拟文件，链接到真实Namespace文件上。exec指令通过setns()加入到某个进程已有的Namespace中，从而达到进入进程容器的目的。

Volume:允许你将宿主机指定目录或文件，挂载到容器里进行读取和修改

```java
docker run -v /home:/test 将home目录挂载到容器/test目录上
```

## docker 层级结构

- rootfs
	- 只读层：from，CMD等指令
	- lnit层：存放临时被修改过的/etc/hosts等文件
	- 可读写层：以copy-on-write方式存放任何对只读层修改，容器声明Volume挂载点，也出现在这一层
	
## Pod

Pod时Kubernetes项目中最基础的一个对象，Pod里的容器共享同一个Network Namespace，同一组数据卷，从而高效率交换信息。

## Service

Kubernetes提供Service服务用于绑定Pod，作为Pod的代理入口，代替Pod对外暴露一个固定网络地址

两个容器间不仅有“访问关系”，如：Web应用对数据库访问时需要Credential信息，Kubernetes提供Secret对象，保存在Etcd里键值对数据。这样Pod启动时，自动把Secret数据以Volume方式挂载到容器里。

Job：描述一次性运行的Pod
DaemonSet：描述每个宿主机上必须且只能运行一个副本的守护进程服务。
CronJob：用于描述定时任务

Kubernetes使用声明式API描述容器化业务和容器间关系的设计思想。按照用户意愿和整个系统规则，完全自动化处理好容器之间各种关系。也就是编排。

若使用容器来部署Kubernetes,那么kubelet的操作宿主机就会变得很麻烦

kubeadm：Kubernetes独立部署工具，把kubelet直接运行宿主机上，然后用容器部署其他Kubernetes组件

```java
// 创建一个Master节点
kubeadm init

// 将一个Node节点加入当前集群中
kubeadm join <Master 节点的 IP和端口>

