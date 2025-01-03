---
date: 2024-11-23 15:20:18
title: k8s笔记
createdTime: 23-05-21 (星期日) 21:17
updateTime: 25-01-03 (星期五) 17:28
---


![image-20230521213417394](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521213417394.png)

![image-20230521213437099](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521213437099.png)

![image-20230521213456098](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521213456098.png)

![image-20230521213544895](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521213544895.png)

## K8S的对象

![img](https://static001.geekbang.org/resource/image/b5/cf/b5a7003788cb6f2b1c5c4f6873a8b5cf.jpg?wh=1920x1298)

### 离线业务->临时任务->只跑一次:job,定时任务:cronjob

**一些字段**
activeDeadlineSeconds，设置 Pod 运行的超时时间。

backoffLimit，设置 Pod 的失败重试次数。

completions，Job 完成需要运行多少个 Pod，默认是 1 个。

parallelism，它与 completions 相关，表示允许并发运行的 Pod 数量，避免过多占用资源。

![image-20230521214214049](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521214214049.png)

![img](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/9b780905a824d2103d4ayyc79267ae28.jpg)![img](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/yy352c661ae37dd116dd12c61932b43c.jpg)

### ConfigMap/Secret:明文/秘文配置信息

> 不是容器 不需要spec字段说明允许规格
>
> containers”里有一个“env”，它定义了 Pod 里容器能够看到的环境变量
>
> 可以使用了简单的“value”，把环境变量的值写“死”在了 YAML 里
>
> 实际上它还可以使用另一个“valueFrom”字段，从 ConfigMap 或者 Secret 对象里获取值
>
> * name”字段是 API 对象的名字，而不是 Key-Value 的名字。

![img](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/0663d692b33c1dee5b08e486d271b69d.jpg)

![image-20230521214405793](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521214405793.png)

![image-20230521214605773](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521214605773.png)

### Deployment:管理pod->pod管理container

#### DaemonSet (spec 里没有 replicas 字段)在**每个节点上只创建出一个 Pod 实例**

如需要运行在master node上 需要设置污点/容忍度 taint:tolerations

![img](https://static001.geekbang.org/resource/image/c1/1c/c1dee411aa02f4ff2b8caaf0bd627a1c.jpg?wh=1920x1173)
#### StatefulSet

> 专门部署“有状态应用”的 API 对象 StatefulSet，它与 Deployment 非常相似，区别是由它管理的 Pod 会有固定的名字、启动顺序和网络标识，这些特性对于在集群里实施有主从、主备等关系的应用非常重要

![img](https://static001.geekbang.org/resource/image/71/88/71b485401dca6946fe4788fa97e3fd88.png?wh=1268x414)

> **所管理的 Pod 不再是随机的名字了，而是有了顺序编号**，从 0 开始分别被命名为 redis-sts-0、redis-sts-1，Kubernetes 也会按照这个顺序依次创建（0 号比 1 号的 AGE 要长一点），这就解决了“**有状态应用”的第一个问题：启动顺序**
>
> **有了这个唯一的名字，应用就可以自行决定依赖关系了**，
>
> 如以 Redis 为例子里，就可以让先启动的 0 号 Pod 是主实例，后启动的 1 号 Pod 是从实例
>
> Service 自己会有一个域名，格式是“对象名. 名字空间”，当我们把 Service 对象应用于 StatefulSet 的时候,**会为 Pod 再多创建出一个新的域名，格式是“Pod 名. 服务名. 名字空间.svc.cluster.local”/“Pod 名. 服务名**”
>
> 外部的客户端只要知道了 StatefulSet 对象，就**可以用固定的编号去访问某个具体的Pod了，虽然 Pod 的 IP 地址可能会变，但这个有编号的域名由 Service 对象维护，是稳定不变的**

![img](https://static001.geekbang.org/resource/image/1a/0f/1a06987c87f3db948b591883a81bac0f.jpg?wh=4000x2946)

### Service与Ingress

> 不再关心 Pod 的具体地址，只要访问 Service 的固定 IP 地址

![img](https://static001.geekbang.org/resource/image/0f/64/0f74ae3a71a6a661376698e481903d64.jpg?wh=1920x1322)

#### ingress:集群内外边界上的入口

> Ingress 的路由规则是 HTTP 协议，所以就不能用 IP 地址的方式访问，必须要用域名、URI

![img](https://static001.geekbang.org/resource/image/e6/55/e6ce31b027ba2a8d94cdc553a2c97255.png?wh=1288x834)

#### 四者联系

![img](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/bb7a911e10c103fb839e01438e184914.jpg)



## 几种更新策略

> 普通部署 全部停掉 然后全部启动新的
>
> 用时最短

![image-20230521220007395](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521220007395.png)

### 滚动更新

![image-20230521220049973](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521220049973.png)

### 蓝绿部署

![](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521220049973.png)

金丝雀发布

![image-20230521220129212](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521220129212.png)

## 应用更健康

> 创建容器有三大隔离技术：namespace、cgroup、chroot。其中的 namespace 实现了独立的进程空间，chroot 实现了独立的文件系统,cgroup 的作用是管控 CPU、内存，保证容器不会无节制地占用基础资源，进而影响到系统里的其他应用

### cgroup

CPU、内存与存储卷,在 Pod 容器的描述部分添加一个新字段 resources 进行申请，它就相当于申请资源的 Claim

![image-20230521221149186](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521221149186.png)

> Kubernetes 里 CPU 的最小使用单位是 0.001，为了方便表示用了一个特别的单位 m，也就是“milli”“毫”的意思，比如说 500m 就相当于 0.5

### 容器状态探针(Probe)

>  除了保证崩溃重启，还必须要能够探查到 Pod 的内部运行状态，定时给应用做“体检”，让应用时刻保持“健康”，能够满负荷稳定工作

* Startup，启动探针，用来检查应用是否已经启动成功，适合那些有大量初始化工作要做，启动很慢的应用。

* Liveness，存活探针，用来检查应用是否正常运行，是否存在死锁、死循环。
* Readiness，就绪探针，用来检查应用是否可以接收流量，是否能够对外提供服务

![img](https://static001.geekbang.org/resource/image/64/d9/64fde55dd2eab68f9968ff34218646d9.jpg?wh=1920x1200)

#### 探测方式

![image-20230521221515092](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521221515092.png)

> 启用了 80 端口，然后用 location 指令定义了 HTTP 路径 /ready，它作为对外暴露的“检查口”，用来检测就绪状态，返回简单的 200 状态码和一个字符串表示工作正常

![image-20230521221701080](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521221701080.png)

![image-20230521221605863](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/ita/image-20230521221605863.png)

### namespace:通过不同命名空间实现资源隔离

> 可以像管理容器一样，给名字空间设定配额，把整个集群的计算资源分割成不同的大小，按需分配给团队或项目使用
