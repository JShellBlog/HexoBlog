# Docker

## 1. Docker 背景

![](https://pic4.zhimg.com/80/5d2cabafe63f33389c8a5c8ae1e576bb_hd.jpg)

### 1.1. Docker 是什么

Docker 是 [PaaS](https://baike.baidu.com/item/PaaS) 提供商 dotCloud 开源的一个基于 [LXC](https://baike.baidu.com/item/LXC) 的**高级容器引擎**，源代码托管在 [Github](https://baike.baidu.com/item/Github) 上, 基于[go语言](https://baike.baidu.com/item/go%E8%AF%AD%E8%A8%80)并遵从Apache2.0协议开源。

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

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

  > Docker容器和文件夹类似，容器包含了应用运行所需到环境。Docker容器可以运行、开始、停止、移动和删除。

# 2.2. 组件

Docker 采用的是客户端/服务端(C/S)架构模式。

- Docker守护进程

处理所有的Docker 请求，管理所有容器。

- Docker客服端

https://blog.csdn.net/zjin_hua/article/details/52041757

## 2.3. 技术基础

- namespace

- cgroups

- unionfs

  https://blog.csphere.cn/archives/22

### 参考资源:

http://www.ruanyifeng.com/blog/2018/02/Docker-tutorial.html

http://www.ruanyifeng.com/blog/2018/02/Docker-wordpress-tutorial.html

https://blog.csphere.cn/archives/22

http://www.Docker.org.cn/book/Docker/what-is-Docker-16.html

http://www.Docker.org.cn/page/resources.html

https://www.zhihu.com/question/22969309

https://www.zhihu.com/question/28300645
