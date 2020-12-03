---
title: Docker初窥+nginx
date: 2020-12-03 15:20:18
tags:
  - 拜读
categories:
  - 分享
---

> 本学期第十四周：



[阮一峰：Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

#### 需求：

`软件运行需要环境配置。换一台机器就要重新配置`

**希望**：**软件可以带环境安装：安装时把运行所需环境复制过来**

已有解决方案：虚拟机，Linux容器

* 虚拟机：独占一部分内存和硬盘空间，运行时其他程序不能使用

* * **资源占用多**：哪怕应用程序真正使用只有1mb，虚拟机也需要几百mb才能运行
  * **多余步骤多**：虚拟机是完整操作系统，无法跳过用户登录等“系统级别操作步骤”
  * **启动慢**：启动速度等于操作系统

* Linux容器

* * **不模拟完整操作系统，而是对进程进行隔离**

  * > 或者说，在正常进程的外面套了一个[保护层](https://opensource.com/article/18/1/history-low-level-container-runtimes)。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。
    >
    > 由于容器是进程级别的，相比虚拟机有很多优势。

  * **启动快**：启动进程而不是启动操作系统

  * **资源占用少**：只占用需要的资源，多个容器可以共享资源，不想虚拟机要独享

  * **体积小**：只包含用到的组件即可，不需要打包整个操作系统

  ### Docker是什么

> **Linux容器的一种封装，提供简单易用的容器使用接口，当前最流行的方案**

* 可以方便的创建和使用容器，把软件放入容器
* 容器可以进行版本管理，复制，分享，修改

#### 用途

* **提供一次性环境**：如测试
* **提供弹性的云服务**：Docker容器可以随开随关，适合动态扩容和缩容
* **组建微服务架构**：一个机器可以通过多个容器跑多个服务，在本机就可以模拟出微服务架构

### 怎么用

#### 安装

> 有社区版和企业版，个人开发者社区版够用
>
> Windows，ubuntu，mac等都有

#### Q&A

Q:

>  Q:不同操作系统上Docker是否有区别，linux上的Docker容器是否可以跑在linux上

A:

> A:未知
>
> 由下文有Ubuntu镜像+apahce
>
> 容器内镜像类似小型虚拟机？与容器外操作系统无关？

##### 安装成功后

* 运行命令验证

```bash
$ docker version
# 或者
$ docker inf
```

* docker需要用户有sudo权限，避免每次输入sudo，可以把用户加入docke用户组

```bash
 sudo usermod -aG docker $USER
```

* Docker 是服务器----客户端架构。命令行运行`docker`命令的时候，需要本机有 Docker 服务。如果服务没有启动，可以用下面的命令启动

```bash
# service 命令的用法
$ sudo service docker start

# systemctl 命令的用法
$ sudo systemctl start docker
```

#### image文件

**应用程序及其依赖，打包在 image 文件里面**

* 通过这个文件，才能生成 Docker 容器

* 可以看作是容器的模板

* 实际开发中，**一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成**。

  > 你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的 image。

  Q：

  >  Ubuntu 的 image不也是一个操作系统？

* 尽量使用别人制作好的，或者基于别人的加工

> 一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

---

### 实操：

* 下载一个镜像并运行
* 停止程序
* 停止服务型容器
* 终止和删除容器文件
* 制作自己的docker容器？镜像
* 其他有用的命令



### 待续

### [Docker微服务教程：架设wordpress网站](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

> - 方法 A：自建 WordPress 容器
> - 方法 B：采用官方的 WordPress 容器
> - 方法 C：采用 Docker Compose 工具

# [Nginx docker容器教程](http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)