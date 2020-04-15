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

Pod使用中间容器Infra，Infra被第一个创建，其他Pod内容器通过Join Network Namespace的方式，和Infra容器关联在一起。

当我们想在一个容器里跑多个功能并不相关应用时，应该优先考虑是不是应该被描述成一个Pod里多个容器

Pod扮演的是传统部署环境里“虚拟机”的角色。凡是**调度，网络，存储，安全相关属性，基本上是Pod级别的**；

**Pod生命周期**
Pod 生命周期的变化，主要体现在 Pod API 对象的Status 部分

**pod.status.phase** 当前状态
- Pending：意味着Pod的YAML文件已提交Kubernetes,API对象已创建保存在Etcd中。但是有些容器没有创建成功
- Running：Pod调度成功，跟具体节点绑定。包含容器创建成功且至少一个运行中
- Succeeded：Pod内所有容器正常运行完毕且已退出，常见于一次性任务
- Failed：Pod内至少有一个容器以不正常状态退出。
- Unknown：Pod状态不能持续被kubelet汇报给kube-apiserver，比如主从节点通信出现问题

### Projected Volume

通过kubectl create ****** 或从文件目录创建 或从编写YAML文件

为容器提供预先定义好的数据。

- Secret：把Pod想要访问的加密数据，存放到Etcd中，通过在Pod容器挂载Volume方式，访问Secret保存的信息。常用于存放数据库的Credential信息。
- ConfigMap：保存不需要加密，应用所需的配置信息。
- Downward API：让Pod里容器能够直接获取这个Pod API对象本身信息，容器进程启动前就能确定的信息。
- Service Account：Kubernetes 系统内置的一种账户服务，是 Kubernetes进行权限分配的对象。Service Account：Kubernetes授权文件和信息，保存绑定在一个Secret对象，也就是ServiceAccountToken。所有运行Kubernetes集群的应用，都必须使用ServiceAccountToken里的授权信息，才能合法访问API Server

### livenessProbe

容器启动后执行指定命令，定时轮询某文件夹，确认容器的健康状况。

### restartPolicy

Pod的恢复机制，pod.spec.restartPolicy标准字段，默认值为Always，任何时候容器发生了异常，一定会被重建。应为Pod与接地那Node绑定，Pod恢复永远发生在当前节点上，

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

```

## 控制器

Deployment是一个基本控制器，通过循环控制流程，创建，更新，删除一些Pod，实现对各种不同对象或资源进行编排操作。