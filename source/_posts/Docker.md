---
title: Docker 简单介绍
date: 2018-06-01 19:59:25
tags: Docker
categories: tools
---

# Docker

## 1. Docker 背景

![](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=2dc9b7759a8fa0ec6bca6c5f47fe328b/2cf5e0fe9925bc31137974de55df8db1cb13704b.jpg)

### 1.1. Docker 是什么

Docker 是 [PaaS](https://baike.baidu.com/item/PaaS) 提供商 dotCloud 开源的一个基于 [LXC](https://baike.baidu.com/item/LXC) (LXC 其并不是一套[硬件虚拟化方法](http://en.wikipedia.org/wiki/Platform_virtualization) 无法归属到全虚拟化、部分虚拟化和半虚拟化中的任意一个，而是一个[操作系统级虚拟化](http://en.wikipedia.org/wiki/Operating_system-level_virtualization)方)的**高级容器引擎**，源代码托管在 [Github](https://baike.baidu.com/item/Github) 上, 基于[go语言](https://baike.baidu.com/item/go%E8%AF%AD%E8%A8%80)并遵从Apache2.0协议开源。

**Docker** 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

![](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=f451b09fa38b87d6444fa34d6661435d/203fb80e7bec54e7719c18b0bb389b504fc26a2f.jpg)

<!--more-->

有人以通俗的方式说明:

> Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送水果的船和专门运送化学品的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都运走。  
> 
> Docker就是类似的理念。现在都流行云计算了，云计算就好比大货轮。Docker就是集装箱。

### 1.2. Docker 的用途

- 快捷部署软件环境

- 以低效耗实现应用资源隔离

- 微服务架构组建（多个Docker 镜像之间组合使用）

- web应用的自动化打包和发布

国内有应用于Docker技术的公司[DaoCloud](http://www.daocloud.io) , [云雀](http://www.alauda.cn)等

## 2. Docker 基础

## 2.1. 术语

- Docker镜像

  > 镜像是Docker 容器运行时的只读模板，每一个镜像由一系列的层（layers）组成。当我们修改镜像时，新的层被创建并透明覆盖之前的层，Docker使用UnionFS 来将这些层连贯到文件系统。

- Docker仓库

  > 类似github， Docker 镜像管理库，Docker官方[Docker Hub](https://hub.docker.com/explore/), 上面有很火镜像资源，如：
  > 
  > ![Nginx](https://hub.docker.com/public/images/official/nginx.png)
  > 
  > ![](https://hub.docker.com/public/images/official/httpd.png)
  > 
  > ![](https://hub.docker.com/public/images/official/ubuntu.png)

- Docker容器

  > 容器都是从镜像建立的，Docker容器和文件夹类似，容器包含了应用运行所需的环境和数据等。Docker容器可以运行、开始、停止、移动和删除。镜像是只读的，当Docker 运行容器时，它会在镜像顶层添加一个可读写的层。

# 2.2. 组件

Docker 采用的是客户端/服务端(C/S)架构模式。Docker客户端和守护进程之间通过socket或者RESTful API进行通信。

![](https://gss2.bdstatic.com/9fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=9f1b2701eddde711f3df4ba4c686a57e/a50f4bfbfbedab644936dac4ff36afc379311e69.jpg)

| 子项         | 说明                                                                       |
|:---------- | ------------------------------------------------------------------------ |
| Docker守护进程 | 建立、运行、发布你的Docker容器，处理所有的Docker 请求，管理所有容器。                                |
| Docker客服端  | Docker客户端，实际上是`docker`的二进制程序，是主要的用户与Docker交互方式。它接收用户指令并且与背后的Docker守护进程通信 |

## 2.3. 技术基础

### 2.3.1. namespace

LXC所实现的隔离性主要是来自kernel的namespace, 其中`pid`, `net`, `ipc`, `mnt`, `uts` 等namespace将container的进程, 网络, 消息, 文件系统和hostname 隔离开。

| NameSpace(ns)  | Function                                                                                                                                                                                                                                                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pid namespace  | 具有如下特征： <br><ul><li> 每个namespace中的pid是有自己的pid=1的进程(类似`/sbin/init`进程) </li><li>个namespace中的进程只能影响自己的同一个namespace或子namespace中的进程 </li><li>因为`/proc`包含正在运行的进程，因此在container中的`pseudo-filesystem`的/proc目录只能看到自己namespace中的进程</li><li>因为namespace允许嵌套，父namespace可以影响子namespace的进程，所以子namespace的进程可以在父namespace中看到，但是具有不同的pid </li></ul> |
| net namespace  | 有了 `pid` namespace, 每个namespace中的pid能够相互隔离，但是网络端口还是共享host的端口。网络隔离是通过`net`namespace实现的， 每个`net` namespace有独立的 network devices, IP addresses, IP routing tables, `/proc/net` 目录。这样每个container的网络就能隔离开来。 LXC在此基础上有5种网络类型，docker默认采用veth的方式将container中的虚拟网卡同host上的一个docker bridge连接在一起。                                               |
| ipc namespace  | container中进程交互还是采用linux常见的进程间交互方法(interprocess communication - IPC), 包括常见的信号量、消息队列和共享内存。然而同VM不同，container 的进程间交互实际上还是host上具有相同pid namespace中的进程间交互，因此需要在IPC资源申请时加入namespace信息 - 每个IPC资源有一个唯一的 32bit ID。                                                                                                                           |
| mnt namespace  | 类似`chroot`，将一个进程放到一个特定的目录执行。`mnt` namespace允许不同namespace的进程看到的文件结构不同，这样每个 namespace 中的进程所看到的文件目录就被隔离开了。同`chroot`不同，每个namespace中的container在`/proc/mounts`的信息只包含所在namespace的mount point。                                                                                                                                            |
| uts namespace  | UTS("UNIX Time-sharing System") namespace允许每个container拥有独立的hostname和domain name, 使其在网络上可以被视作一个独立的节点而非Host上的一个进程。                                                                                                                                                                                                                  |
| user namespace | 每个container可以有不同的 user 和 group id, 也就是说可以以container内部的用户在container内部执行程序而非Host上的用户                                                                                                                                                                                                                                                |

有了以上6种namespace从进程、网络、IPC、文件系统、UTS和用户角度的隔离，一个container就可以对外展现出一个独立计算机的能力，并且不同container从OS层面实现了隔离。 然而不同namespace之间资源还是相互竞争的，仍然需要类似`ulimit`来管理每个container所能使用的资源 - LXC 采用的是`cgroup`。

### 2.3.2. Control Groups(cgroups)

**cgroups** 实现了对资源的配额和度量。 **cgroups**  的使用非常简单，提供类似文件的接口，在 `/cgroup`目录下新建一个文件夹即可新建一个group，在此文件夹中新建**task**文件，并将pid写入该文件，即可实现对该进程的资源控制。
我们主要关心cgroups可以限制哪些资源，即有哪些subsystem是我们关心。

**`cpu`**: 在cgroup中，并不能像硬件虚拟化方案一样能够定义CPU能力，但是能够定义CPU轮转的优先级，因此具有较高CPU优先级的进程会更可能得到CPU运算。 通过将参数写入**cpu.shares**,即可定义改cgroup的CPU优先级 - 这里是一个相对权重，而非绝对值。当然在cpu这个subsystem中还有其他可配置项，手册中有详细说明。

**`cpusets`** : cpusets 定义了有几个CPU可以被这个group使用，或者哪几个CPU可以供这个group使用。在某些场景下，单CPU绑定可以防止多核间缓存切换，从而提高效率

**`memory`** : 内存相关的限制

**`blkio`** : block IO相关的统计和限制，byte/operation统计和限制(IOPS等)，读写速度限制等，但是这里主要统计的都是同步IO

**`net_cls`**， **`cpuacct`** , **`devices`** , **`freezer`** 等其他可管理项。

### 2.3.3. LinuX Containers(LXC)

借助于namespace的隔离机制和cgroup限额功能，LXC提供了一套统一的API和工具来建立和管理container, LXC利用了如下 kernel 的features:

- Kernel namespaces (ipc, uts, mount, pid, network and user)
- Apparmor and SELinux profiles
- Seccomp policies
- Chroots (using pivot_root)
- Kernel capabilities
- Control groups (cgroups)

LXC 向用户屏蔽了以上 kernel 接口的细节, 提供了如下的组件大大简化了用户的开发和使用工作:

- The liblxc library
- Several language bindings (python3, lua and Go)
- A set of standard tools to control the containers
- Container templates

### 2.3.4. AUFS

Docker对container的使用基本是建立在LXC基础之上的，然而LXC存在的问题是难以通过标准化的模板制作、重建、复制和移动 container。VM虚拟化可以采用image和snapshot 实现复制、重建以及移动的功能。docker0.7中引入了storage driver, 支持AUFS, VFS, device mapper, 也为BTRFS以及ZFS引入提供了可能。 但除了AUFS都未经过dotcloud的线上使用。

AUFS (AnotherUnionFS) 是一种 `Union FS`。AUFS支持为每一个成员目录(AKA branch)设定'readonly', 'readwrite' 和 'whiteout-able' 权限,

典型的Linux启动到运行需要两个FS - bootfs + rootfs。

![](https://gss3.bdstatic.com/-Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=fb52a2bc700e0cf3b4fa46a96b2f997a/9358d109b3de9c82ba58f6216e81800a19d8435a.jpg)



典型的Linux在启动后，首先将 rootfs 置为 readonly, 进行一系列检查, 然后将其切换为 "readwrite" 供用户使用。在docker中，起初也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的将一个 readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 file system设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行目录, 每一个被称作一个Layer。如下图:

![](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=2a7eeea18594a4c21e2eef796f9d70b0/54fbb2fb43166d22b3bc1a3a442309f79152d251.jpg)得益于AUFS的特性, 每一个对readonly层文件/目录的修改都只会存在于上层的writeable层中。这样由于不存在竞争, 多个container可以共享readonly的layer。 所以docker将readonly的层称作"**image**"`- 对于container而言整个rootfs都是read-write的，但事实上所有的修改都写入最上层的writeable层中, image不保存用户状态，可以用于模板、重建和复制。

由此可见，采用AUFS作为docker的container的文件系统，能够提供如下好处:

1. 节省存储空间 \- 多个container可以共享base image存储

2. 快速部署 \- 如果要部署多个container，base image可以避免多次拷贝

3. 内存更省 \- 因为多个container共享base image, 以及OS的disk缓存机制，多个container中的进程命中缓存内容的几率大大增加

4. 升级更方便 \- 相比于 copy-on-write 类型的FS，base-image也是可以挂载为可writeable的，可以通过更新base image而一次性更新其之上的container

5. 允许在不更改base-image的同时修改其目录中的文件 - 所有写操作都发生在最上层的writeable层中，这样可以大大增加base image能共享的文件内容。

### 参考资源:

[Docker 百度百科](https://baike.baidu.com/item/Docker/13344470?fr=aladdin)

[Docker Org Docs](https://docs.docker.com/)

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
[Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

[一小时Docker教程](https://blog.csphere.cn/archives/22)

[Docker Getting Start: Related Knowledge](http://tiewei.github.io/cloud/Docker-Getting-Start/)

[非常详细的 Docker 学习笔记](https://blog.csdn.net/zjin_hua/article/details/52041757)

[docker 中文](http://www.docker.org.cn/book/Docker/what-is-Docker-16.html)

[Docker资源](http://www.docker.org.cn/page/resources.html)

[如何通俗解释Docker是什么？](https://www.zhihu.com/question/28300645)
