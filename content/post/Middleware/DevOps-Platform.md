---
title: "DevOps：运维和平台篇"
date: 2023-03-10T18:05:59+08:00
categories:
- 计算机科学与技术
- DevOps
tags:
- DevOps
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/DevOps.jpg
---
本篇讲述DevOps相关内容中，和运维平台相关的内容。
<!--more-->
## 运维历史
## 常见中间件
1. Docker：
    - 架构：
        - docker client：控制行命令工具
        - docker daemon：docker执行的server
        - registry：image store
        - containerd
        - runc
        - bundle
        - layer
        - http2p agent、server
    - 原理：linux namespace + linux cgroups + rootfs构建进程隔离环境
        - 静态视图：一组联合挂载在docker-imges上的rootfs，即容器镜像
        - 动态视图：由namespace+cgroups构成的隔离环境，即容器运行时
    - 资源隔离细节：
        - cgroups（control groups）：主要隔离进程的cpu和内存资源。原理是为进程加上一些钩子（hook，检测到对应消息就先于消息处理函数执行），当任务执行涉及到某个资源时就会触发钩子上附带的subsystem进行检查，根据不同资源使用对应技术进行资源限制和优先级分配。
            - 补充知识，linux的进程结构体task_struct，进程状态，结合操作系统看进程状态，状态中还涉及到异步信号
            - 核心结构体cgroupfs_root，cgroup_subsys，一个子系统只能挂载一个
        - cpu共享原理：
            - 调度器组成：主调度器、周期性调度器、调度器类（CFS是其一种实现）。
                - 调度器分为主调度器和周期性调度器两种，两种共同组成核心调度器（core ~）或称通用调度器（generic ~）。分别服务不同的场景，比如线程主动让出控制权，或者是周期性检测查看是否有必要更换
                - 周期性调度器、主调度器，内部具体分为实时进程调度和普通进程调度，策略不同，普通一般使用CFS选用红黑树最左侧节点，实时进程一般使用FIFO、RR策略
            - CFS调度器：红黑树结构，优先级分两层（nice值控制一个task的权重，share值控制一个group的权重），整合task_group作为一个sched_entity进行调度，优先级高的vruntime跑的慢
            - cpu quota：share方式只能控制容器间负载拉满时的一种下限。再引入cfs_bandwidth，并结合task_group进行限制。也相当于vruntime，整个组针对物理机的核数做一个限制。
        - 内存和I/O隔离：
            - 内存OOM：系统OOM，或者cgroup级别OOM。
            - 磁盘I/O：缓存和脏页回写。bdi_writeback，每个磁盘都会有一个线程，专门负责磁盘的page cache数据刷新工作。目前隔离做的不够好，没办法知道回写由哪个cgroup发起。某一个进程打满磁盘会影响其他进程。
    - 视图隔离：
        - namespace：提供了虚拟化的轻量级实现，部分数据只能局部可见
        - 类型：uts、ipc、mnt、pid、net、cgroup（以这些内容为代表划分可见性）
        - 常用的是PID namespace：父NS可看到子NS的全部信息，反之不行。每个NS必须有一个进程，充当init进程，（实际使用systemd），如果它挂了，就是容器程序挂了，和物理机一样。
    - 文件系统隔离：
        - 原理：Image和Union Mount技术
        - oci image：image layer filesystem changeset。包含文件和变更，序列化为blob文件；包含manifest表示内容和依赖等meta信息；包含configuration表示启动所用的指令、参数等。
        - 实际运行镜像时，image内容可读不可写，还会有另外的layer负责可读可写，但会在容器退出时删除。
        - union mount：（overlayfs、overlayfs2、aufs等具体技术）将多个目录（image内的内容）联合挂载在一个路径下，文件仍在本地系统上。用chroot系统调用替换为子系统根。
        - CI/CD的最后 打包输出，很有可能就是一个docker镜像，并进行后续的registry，分发等。
    - 一次容器创建过程：(注意image展开后，用bundle配置相关目录)
        1. docker client发起docker run指令
        1. docker host上的docker daemon接收到http请求， 由http2p agent拉取docker镜像到本地
        1. dockerd挂载容器实例根路径rootPath、配置bundle目录
        1. dockerd通过grpc调用containerd创建容器实例
        1. containerd执行runc
        1. runc读取bundle下配置，以挂载路径rootPath作为rootfs启动容器实例
    - 监控：eBPF（KProbe） based metrics，非侵入式。（选择要探测的内核函数，eBPF调用kProbe，写入map，用户态读取map，将进程级数据聚合到cgroup级别）

1. OpenStack
1. Kubemetes
    - k8s简单介绍：支持灵活的调度机制、丰富的组件元语（支持无状态、有状态服务的组件元语，方便扩缩容）、支持高性能的服务注册/订阅功能、扩展性强支持插件化二次开发。
    - service port：集群内部应用访问方式。node port：集群外部服务访问方式。k8s通过改写相关iptable支持请求转发到相关pod。
    - 调度机制：亲和性和反亲和性（出于容灾考虑）。【基于pod的label进行匹配】

## 总结
&emsp;&emsp;软件开发行业需要解决的若干问题总而言之就是：提高代码和组件分发、开发、合并、测试、部署上线、回滚的效率。

## 参考
1. [Linux核心调度器之周期性调度器scheduler_tick--Linux进程的管理与调度(十八） ](https://www.cnblogs.com/linhaostudy/p/9867364.html)
1. [Linux系统核心调度器——主调度器schedule函数详解](https://blog.csdn.net/weixin_42092278/article/details/88778435?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)