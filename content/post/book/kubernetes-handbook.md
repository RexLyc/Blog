---
title: "Kubernetes读书笔记总结"
date: 2024-03-10T21:50:41+08:00
categories:
- 读书笔记
- 技术书
tags:
- Kubernetes
- 中间件
- 施工中
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
16. 容器：容器是包含在任何环境中运行所需的所有元素的软件包。是软件的可执行单元。
16. 容器运行时（Container Runtime）是指管理容器的一类软件组件。它提供了一种隔离和管理应用程序的环境，使得应用程序可以在独立的环境中运行，而不会相互干扰。
17. rkt：和docker同类的，另一种容器运行时实现。
18. sidecar容器：一个与应用容器运行在同一个Pod上的容器。常见例子有：日志输送、日志观察者、监控代理等等。

## 学习准备
搭建K8s运行环境，[参考官方文档](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/)。本文由于是学习，所以并没有真正的集群用于搭建。在本地电脑上运行的（CentOS7）。

如果是在本地电脑上运行，需要安装Minikube，Minikube是一个在本机上用单节点（只有控制平面节点）模拟集群的环境，[安装说明](https://minikube.sigs.k8s.io/docs/start/)
直接进行minikube的搭建比较困难，如果可以的话，建议先用docker，将所需镜像拉下来
```bash
# 直接start大概率起不来，因为镜像下不到
# minikube start

# 这几个用代理就可以解决
docker pull gcr.io/k8s-minikube/kicbase:v0.0.42
docker pull gcr.io/k8s-minikube/storage-provisioner:v5

# 但还有几个报错的，即使使用了代理，还是可能有网络问题,报错如下
# ! The image 'registry.k8s.io/kube-apiserver:v1.28.3' was not found; unable to add it to cache.
# 此时建议直接使用镜像下载
docker pull k8s.mirror.nju.edu.cn/kube-apiserver:v1.28.3
docker pull k8s.mirror.nju.edu.cn/kube-scheduler:v1.28.3
docker pull k8s.mirror.nju.edu.cn/kube-controller-manager:v1.28.3
docker pull k8s.mirror.nju.edu.cn/pause:3.9
docker pull k8s.mirror.nju.edu.cn/kube-proxy:v1.28.3
docker pull k8s.mirror.nju.edu.cn/etcd:3.5.9-0
docker pull k8s.mirror.nju.edu.cn/coredns/coredns:v1.10.1

# 此时需要更换标签，否则minikube依然找不到自己想要的镜像
docker tag k8s.mirror.nju.edu.cn/kube-apiserver:v1.28.3 registry.k8s.io/kube-apiserver:v1.28.3
docker tag k8s.mirror.nju.edu.cn/kube-scheduler:v1.28.3 registry.k8s.io/kube-scheduler:v1.28.3
docker tag k8s.mirror.nju.edu.cn/kube-controller-manager:v1.28.3 registry.k8s.io/kube-controller-manager:v1.28.3
docker tag k8s.mirror.nju.edu.cn/pause:3.9 registry.k8s.io/pause:3.9
docker tag k8s.mirror.nju.edu.cn/kube-proxy:v1.28.3 registry.k8s.io/kube-proxy:v1.28.3
docker tag k8s.mirror.nju.edu.cn/etcd:3.5.9-0 registry.k8s.io/etcd:3.5.9-0
docker tag k8s.mirror.nju.edu.cn/coredns/coredns:v1.10.1 registry.k8s.io/coredns/coredns:v1.10.1

# 之后再启动
minicube start --driver=docker
```

> 搭建过程中可能遇到网络问题，建议选定[镜像](https://www.cnblogs.com/wswind/p/14420803.html)，如果仍有问题，可以参考[代理配置](https://blog.csdn.net/qq_22903841/article/details/123106604)，配置文中dockerd的\[Service\]，并重启即可。

> 在配置了代理的情况下，前述的脚本已经能工作。

注意Minicube虽然使用了docker，但是由于其本身在运行时，有自己的镜像仓库，因此在minikube内，使用k8s部署容器时，需要先将镜像引入到minikube中。例如
```bash
docker pull nginx
minikube image load nginx
```
此后可以直接在k8s的yaml配置中，使用nginx，并使用```imagePullPolicy: Never```来指示使用本地镜像。

在基础搭建之外，可以考虑一些必要的插件。
1. 添加Minikube中的Web页面Dashboard：[部署和访问 Kubernetes 仪表板（Dashboard）](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/)。注意按照配置走，然后URL可能要修改一下
    ``` bash
    # 需要修改一下proxy的参数，否则会对访问来源有限制，这一步将8001端口暴漏出去
    kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'

    # 访问地址
    http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
    ```

## 基本使用
创建pod的示例，来自于网络
```yaml
# shiro是apache的一个登录验证的库
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shiro550
spec:
  selector:
    matchLabels:
      app: shiro550
  replicas: 1
  template:
    metadata:
      labels:
        app: shiro550
    spec:
      containers:
        - name: shiro550
          imagePullPolicy: IfNotPresent
          image: vulhub/shiro:1.2.4
          ports:
            - containerPort: 8080
              name: web     
```

以下示例[来自网络](https://blog.csdn.net/qq_34168515/article/details/119893456)
```yaml
# 先提交一份configMap
apiVersion: v1
kind: ConfigMap
metadata:
    name: web-nginx-config
data:
  nginx.conf: |
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;


    events {
        worker_connections  1024;
    }


    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;

        #gzip  on;

        include /etc/nginx/conf.d/*.conf;
    }


# 再提交一个使用configMap的Nginx
```

使用Service创建内部网络接口
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
# 作用于带有run: my-nginx标签的pod
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
# 作用于带有run: my-nginx标签的pod
  selector:
    run: my-nginx
```

下面列举一下kubectl的指令
```bash
kubectl get pods
kubectl get nodes
kubectl addon list
kubectl proxy
kubectl port-forward
```


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

相比之下CNI 仅关心容器创建时的网络分配，和当容器被删除时释放网络资源。CNI接口定义只有四个方法，添加/删除网络，添加/删除网络列表。这个接口的实现就是CNI插件。CNI插件以可执行文件的形式被实现。负责将网络接口插入到容器的网络命名空间。并在宿主侧进行所有必要的改变。然后将 IP 分配给接口，并通过调用适当的 IPAM 插件来设置与“IP 地址管理”部分一致的路由。整体来说作用就是：将容器添加到网络、从网络中删除容器。

![CSI架构](/images/book/k8s/csi-arch.webp)
上图所示的是CSI相关架构，尤其侧重展示了对CSI插件的部署方式（StatefulSet、Daemonset）。CSI是容器存储接口， 试图建立一个行业标准接口的规范。借助 CSI ，容器编排系统（CO）可以将任意存储系统暴露给自己的容器工作负载。通过CSI接口，存储提供商可以编写和部署插件，而不是修改核心的K8s代码。CSI接口的两端分别是Kubelet、CSI驱动程序。Kubelet通过Unix域套接字直接向CSI驱动程序发起CSI调用（例如NodeStageVolume，NodePublishVolume等），以挂载和卸载卷。也就是k8s官方维护的一系列external组件，负责注册CSI driver 或监听k8s对象资源，从而发起csi driver调用。

而站在第三方CSI组件开发者的角度，则主要需要实现三个接口：
- Identity Service：用于 Kubernetes 与 CSI 插件协调版本信息
- Controller Service：用于创建、删除以及管理 Volume 存储卷
- Node Service：用于将 Volume 存储卷挂载到指定的目录中以便 Kubelet 创建容器时使用（需要监听在 /var/lib/kubelet/plugins/[SanitizedCSIDriverName]/csi.sock）

### Pod
Pod 中封装着应⽤的容器（有的情况下是好⼏个容器），存储、独⽴的⽹络 IP，管理容器如何运⾏的策略选项。Pod 代表着部署的⼀个单位：kubernetes 中应⽤的⼀个实例，可能由⼀个或者多个容器组合在⼀起共享资源。使用多个容器的组合是一种比较高级的方式，往往意味着这几个容器必须在一起紧密配合工作。

注意Pod 中共享的环境包括 Linux 的 namespace、cgroup 和其他可能的隔绝环境，这⼀点跟 Docker 容器⼀致。但在 Pod 的环境中，每个容器中可能还有更⼩的⼦隔离环境。

不同 Pod 之间的容器具有不同的 IP 地址，不能直接通过 IPC 通信。

Controller 可以创建和管理多个 Pod，提供副本管理、滚动升级和集群级别的⾃愈能⼒。例如，如果⼀个 Node 故障，Controller 就能⾃动将该节点上的 Pod 调度到其他健康的 Node 上。


### 服务、负载均衡和联网
当拥有了组装pod的能力之后，还需要将pod之间连接起来，以提供完整的服务。在这一步，主要有两种方式。
1. Service：
2. Ingress：

对于Service。Kubernetes 支持两种查找服务的主要模式：环境变量和 DNS。前者开箱即用，而后者则需要 CoreDNS 集群插件。

具体操作指南：[使用Service连接到应用](https://kubernetes.io/zh-cn/docs/tutorials/services/connect-applications-service/)

## 插件
Kubernetes灵活的一点就是支持非常多的插件。


## 附属内容
Istio

## 参考
1. [Kubernetes 基础教程](https://lib.jimmysong.io/kubernetes-handbook/)
2. [Kubernetes 官方中文文档](https://kubernetes.io/zh-cn/docs/home/)
3. [Kubernetes(k8s)是什么？架构是怎么样的？6分钟快速入门](https://www.bilibili.com/video/BV1Du4m137pK/)
4. [K8S CSI容器存储接口(一)：介绍以及原理](https://cloud.tencent.com/developer/news/731936)
5. [Kubernetes指南](https://kubernetes.feisky.xyz/extension/volume/csi)
6. [K8S面试题（史上最全 + 持续更新）](https://www.cnblogs.com/crazymakercircle/p/17052058.html)
7. [K8S 中 Ingress 和 Service 的区别？](https://www.cnblogs.com/Skybiubiu/p/17325021.html)
8. [k8s 快速部署 nginx 并通过 configMap配置 nginx.conf](https://blog.csdn.net/qq_34168515/article/details/119893456)