---
title: "Kubernetes读书笔记总结"
date: 2024-03-10T21:50:41+08:00
categories:
- 读书笔记
- 技术书
tags:
- Kubernetes
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/k8s-logo.png
draft: true
---
本文记录《Kubernetes基础教程》等K8s相关书籍的读书笔记。
<!--more-->

> 本文在编写时，学习所使用的Docker版本为1.24。

## 术语和演进
1. 云原⽣应⽤程序被设计为在平台上运⾏，并设计为拥有弹性，敏捷性，可操作性，易扩展性和可观察性。具备故障隔离保护、不中断业务持续更新的能力。实现过程中通常需要：微服务、健康报告、遥测数据、弹性、声明式（而非命令式）。
2. 演进路线：非虚拟化、虚拟化、IaaS、Paas、2010开源Iaas（OpenStack）、2011开源Paas、2013容器、2015云原生
3. CNCF（Cloud Native Computing Foundation）：云原生计算基金会。
4. 12因素应用（12-Factor app method）：基准代码仓库、显式声明包依赖、配置代码分离、后端服务、CI/CD、无状态进程、端口绑定、并发、快速启停、开发和线上等价、日志、管理进程。
    > 此外还考虑：认证授权、监控、API优先
6. CNI（Container Network Interface）：容器网络接口，提供网络资源
7. CSI（Container Storage Interface）：容器存储接口，提供存储资源
8. CRI（Container Runtime Interface）：容器运行时的对外接口，提供计算资源。CRI实现了k8s和具体容器的解耦，通过CRI，k8s可以不只支持docker，也可以使用其他容器运行时。实现了CRI接口的容器运行时，被称之为CRI shim，上一个gRPC Server。
9. OCI（Open Container Initiative）：容器运行时标准，是容器在运行时和系统之间的接口，真正创建容器。
10. Pod：k8s中的最小管理单元，是一个共享context的组，设计上可以有多个容器在其中共享context并运行。受业务需要，实际分为Deployment、Job、DaemonSet、StatefulSet，分别对应长期、批处理、后台支撑、有状态应用四种业务需要。
11. RC（Replication Controller）：较早的副本控制器，保证Pod高可用的API对象，监控Pod数量，如果不足，就启动新的Pod副本，多了就杀死。
12. RS（Replica Set）：副本集，新一代RC。
13. Service：服务对象。RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量。服务对象解决的是如何访问Pod的问题。客户实际上访问Service，由Service去做反向代理连接具体的Pod。
14. StatefulSet：适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。对于这些Pod，他们是有固定名字的，而且如果一个挂掉，重启之后仍需要使用同样的名字，挂载和之前同样的存储，使用同样的context。
15. 控制平面组件（Control Plane Components）：控制平面组件会为集群做出全局决策，比如资源的调度。 以及检测和响应集群事件，例如当不满足部署的 replicas 字段时，要启动新的 Pod）包括Control Manager、ETCD、Schedulor、API-Server。。控制平面组件可以在集群中的任何节点上运行。 然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件，并且不会在此计算机上运行用户容器。
16. rkt：和docker同类的，另一种容器运行时实现。

## 学习准备
搭建K8s运行环境，[参考官方文档](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/)。

如果是在本地电脑上运行，需要安装Minikube，[安装说明](https://minikube.sigs.k8s.io/docs/start/)

minikube的搭建比较困难，如果可以的话，建议先用docker，将所需镜像拉下来
```bash
docker pull gcr.io/k8s-minikube/kicbase:v0.0.42
docker pull gcr.io/k8s-minikube/storage-provisioner:v5
# 还有几个报错的，先记在这里
# ! The image 'registry.k8s.io/kube-apiserver:v1.28.3' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/kube-scheduler:v1.28.3' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/kube-controller-manager:v1.28.3' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/pause:3.9' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/kube-proxy:v1.28.3' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/etcd:3.5.9-0' was not found; unable to add it to cache.
# ! The image 'registry.k8s.io/coredns/coredns:v1.10.1' was not found; unable to add it to cache.
```

> 搭建过程中可能遇到网络问题，建议选定[镜像](https://www.cnblogs.com/wswind/p/14420803.html)，如果仍有问题，可以参考[代理配置](https://blog.csdn.net/qq_22903841/article/details/123106604)，配置文中dockerd的\[Service\]，并重启即可。

## 核心组件
![Kuberentes 架构（图片来自于网络）](/images/book/k8s/kubernetes-high-level-component-archtecture.jpg)
上图所示的是k8s 架构。可以看见其分为Master、Node。其中
- Master又分为etcd、controller、API、schedulor。
- Node可以是一个虚拟机，也可以是物理机器。Node上运行多个执行具体任务的Pod。同时Node还具备一些必要的组件，如kubelet、kube-proxy等。

### ETCD
使用Raft一致性算法来实现的，分布式一致性KV存储。提供的是通用的存储方案，但目前主要用于共享配置和服务发现。

### API对象（Kubernetes对象）
API设计是云计算系统的核心设计。每个功能都会针对想的引入API对象。每个 API 对象都有 3 大类属性：元数据 metadata、规范 spec 和状态 status。从该对象的设计也可以看出，K8s最核心的思想之一，就是声明而非控制。通过API对象，用户告知系统，希望达到的期望状态（Desired State）。

元数据是用来标识 API 对象的，每个对象都至少有 3 个元数据：namespace，name 和 uid。除此以外还有各种各样的标签 labels 用来标识和匹配不同的对象，例如用户可以用标签 env 来标识区分不同的服务部署环境，分别用 env=dev、env=testing、env=production 来标识开发、测试、生产的不同服务。

规范描述了用户期望 Kubernetes 集群中的分布式系统达到的理想状态（Desired State），例如用户可以通过复制控制器 Replication Controller 设置期望的 Pod 副本数为 3.

status 描述了系统实际当前达到的状态（Status），例如系统当前实际的 Pod 副本数为 2；那么复制控制器当前的程序逻辑就是自动启动新的 Pod，争取达到副本数为 3。

### 开放接口
术语章节中提到的CRI、CNI、CSI是主要的开放接口。

![CRI架构](/images/book/k8s/cri-architecture.png)
如上图所示，CRI是定义容器和镜像提供的服务的接口。因为容器和镜像的生命周期明显不同，实现上分为 RuntimeService 和 ImageService。

相比之下CNI 仅关心容器创建时的网络分配，和当容器被删除时释放网络资源。

## 插件
Kubernetes灵活的一点就是支持非常多的插件。

## 参考
1. [Kubernetes 基础教程](https://lib.jimmysong.io/kubernetes-handbook/)
2. [Kubernetes 官方中文文档](https://kubernetes.io/zh-cn/docs/home/)