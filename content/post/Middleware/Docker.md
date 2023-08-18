---
title: "Docker：All-In-One"
date: 2023-07-31T15:56:50+08:00
categories:
- 计算机科学与技术
- DevOps
tags:
- DevOps
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/docker.jpg
draft: true
---

<!--more-->
## 基础架构：
![Docker Overview](/images/system/docker-arch.svg)
<center>图片来源：官方架构图</center>

术语：
- docker client：控制行命令工具
- docker daemon：docker执行的server。但目前docker daemon已经不会再直接管理&启动容器，而是集成containerd、runc等组件，完成容器管理。
- registry：image store镜像商城
- containerd：负责创建容器，[参考：一文搞懂容器运行时 Containerd](https://www.qikqiak.com/post/containerd-usage/)
- runc：在创建容器过程中需要做的有关namespaces、cgroups的配置，挂在文件系统等操作，这些操作成为OCI（开放容器标准），runc就是这个标准的一种实现。
- bundle：一个根文件系统（rootfs，用于挂载到容器运行时根目录）和一个config.json（容器的配置信息），有了这两样内容，就可以启动一个容器了
- layer：docker所用的镜像实际上是由多个部分组成，每一个称为一个layer，它们都是只读的，每一层仅包含差异部分，按顺序加载这些，最后会有一个可读写层。[参考：Docker 进阶之镜像分层详解](https://developer.aliyun.com/article/981453)
- http2p agent、server
- image：docker镜像，是静态的资源。包含了一个容器所需的所有运行时内容（裁剪的操作系统，程序依赖库，程序，环境变量）
## docker和虚拟机的对比
虚拟机
- 独立性更强，不同虚拟机内部可以使用不同的依赖软件版本
- 速度更慢，尤其是启动
- 需要完整的系统镜像
- 硬件资源密集分配

docker
- 轻量化、直接使用宿主机所用的系统（内核）
- 需要更少的硬件资源

## 原理概述
原理：linux namespace + linux cgroups + rootfs构建进程隔离环境
- 静态视图：一组联合挂载在docker-imges上的rootfs，即容器镜像
- 动态视图：由namespace + cgroups构成的隔离环境，即容器运行时

## 资源隔离
### namespace
&emsp;&emsp;Linux下通过namespace实现资源之间的隔离。namespace并不是在某个版本一次性完成实现的，在不同的内核版本中有不同的开发情况。但他们的目的是相同的，就是希望在不同的资源方面，每个进程都能有自己所属的命名空间。不同命名空间的资源，在需要的时候可以相互隔离，不可见。以下是常见的namespace。推荐阅读[Containers unplugged: Linux namespaces - Michael Kerrisk](https://www.youtube.com/watch?v=0kJPa-1FuoI)
  
  | 名称 | 宏名称 | 隔离资源 |
  | --- | --- | --- |
  | IPC | CLONE_NEWIPC | 进程通信资源：信号量、消息队列、共享内存 |
  | PID | CLONE_NEWPID | 进程编号 |
  | Network | CLONE_NEWNET | 网络设备、网络栈、端口 |
  | Mount | CLONE_NEWNS | 文件系统挂载点 |
  | User | CLONE_NEWUSER | 用户和用户组 |
  | UTS（Unix Time-Sharing System，Unix分时操作系统） | CLONE_NEWUTS | 主机名和域名 |
  | CGroup | CLONE_NEWCGROUP | cgroup根目录 |
  
  > 注1：Mount命名空间的宏名称很特殊，是New Namespace的含义，这是因为它就是实现的第一个命名空间。

  > 注2：伪文件系统/proc里存储了namespace信息，对于每一个进程pid，都能在/proc下找到，/proc/pid/ns/，该文件夹下存储了指向各个命名空间的文件。

  > 注3：/proc/pid/ns/文件夹下采用文件的形式存储命名空间信息，可以很方便的追踪打开这个文件的进程，当某个命名空间下的所有进程都退出，则该文件被关闭。

不同命名空间的隔离意义，参考
1. UTS：隔离主机名和域名，则每个容器可以作为一个主机独立存在，可以视为网络上的一个独立节点，而不是宿主机上的一个进程。当一个进程使用了UTS命名空间隔离，在该进程内调用```sethostname```，不会影响宿主机名称。
2. IPC：同一个IPC命名空间下的进程，可以看到相同的IPC资源，如信号量、消息队列等。
3. PID：重要也复杂的隔离。操作系统为每一个PID命名空间维护一个树状结构，根节点等价于是该PID树的init进程。在这种树状结构中，子命名空间的各节点对父命名空间可见，反之则不行。因此可知：
    - 宿主机上实际可以看到所有容器内进程，但容器内不能所属PID命名空间外部的任何进程。
    - 每一个进程都可能有多个PID，具体应当使用的PID需要参考所处的PID命名空间
    - 系统启动时会创建一个根PID命名空间，所有进程都处于该命名空间下
    - getpid调用返回值，和进程所处pid命名空间相关
    - PID命名空间只能在clone时进行指定（或者说创建时确定），而不能通过setns进行更改。因此PID命名空间只能在新进程中隔离，当前进程是无法切换PID命名空间的。
4. Mount：进行挂载点隔离，可以做到不同命名空间内的进程，其对文件系统的挂载互相不可见。创建时的挂载规则包含私有、共享、主从等规则。在创建时会复制原有挂载信息，并存储针对挂载对象的规则。随后两个Mount命名空间将互相独立。
5. Network：网络隔离，则不同命名空间内的进程，各自拥有独立的网络设备接口、网路协议栈、路由表、防火墙规则等资源。宿主机通过veth建立虚拟网络资源。
6. User：重要也复杂的隔离。对用户、用户组、权限的隔离。该命名空间也具有类似PID命名空间一样的层次结构。一个用户可以在不同的命名空间中，表现出不一样的权限。参考[Linux Namespace：user(第一部分)](https://www.testerfans.com/archives/linux-namespace-user-first-section)。推荐阅读[Containers unplugged: understanding user namespaces - Michael Kerrisk](https://www.youtube.com/watch?v=73nB9-HYbAI)。重点在于理解以下几点：
    - 简单来说，原有的Linux权限模型有安全问题：有些程序往往需要（通过setuid方式）获取root权限来完成相应工作，但是这些程序并不能保证其具备充分的安全性。因此Linux提出了Capabilities模型来描述进程/文件相对应的权限，来拆分原本通通杂糅在root的各种权限（修改密码、修改时钟等）。
    - 创建新的user命名空间后，需要对其进行uid/gid的映射，以明确当前命名空间用户id和父user命名空间中的某个用户id的映射关系。
    - **核心**：所有其他非user命名空间资源，都会和一个user命名空间绑定。即该user命名空间拥有这个资源。当一个进程申请使用资源时，会检查其所处user命名空间，是否拥有该非user命名空间所管理的资源。
    - 子user命名空间中的用户，即使具备root权限，也只是相对于该命名空间所拥有的各种其他命名空间资源具备了root权限。对于从父User命名空间继承而来的资源，并不一定具备root权限（取决于该用户在父命名空间中所具备的权限）。
    - 综上，目前的权限检查是Capabilities和User命名空间的双重检查。
    - Docker默认并不会新建user命名空间，而是会选择继承宿主环境。
7. Cgroup：Control Group，目前有v1，v2两个版本。目标是控制进程对设备的使用，如cpu、内存、块设备（硬盘）等。参考[Linux资源管理之cgroups简介](https://tech.meituan.com/2015/03/31/cgroups.html)


核心系统调用
1. clone：```int clone(int (*fn)(void *), void *child_stack, int flags, void *arg);```
  是fork的一种包装，在创建新进程fn时，传递给其栈child_stack，同时可以用各种CLONE开头的宏来指定flags。
1. setns：```int setns(int fd, int nstype);```
  fd是一个指定的namespace的文件描述符（/proc/xxxx/ns下的某个链接），nstype是所属的namespace类型。通过这个函数，可以将当前进程的namespace进行修改。但setns能修改的命名空间是很有限的，**user、pid均不允许**。
1. unshare：```int unshare(int flags);```
  flags是所指定的namespace类型，调用该函数将会创建并切换到指定的一系列namespace下

### cgroups细节
cgroups（control groups）：主要用于管理、隔离系统硬件的物理资源，如cpu、内存、磁盘IO。核心原理是为进程加上一些钩子（hook，检测到对应消息就先于消息处理函数执行），当任务执行涉及到某个资源时就会触发钩子上附带的subsystem进行检查，根据不同资源使用对应技术进行资源限制和优先级分配。

术语
- 任务（task）：表示一个调度单位，本质上就是一个进程或线程。
- 控制组（cgroup）：资源控制的最小单位。一个任务需要加入一个或多个cgroup才能接收到资源控制
- 子系统（subsystem）：资源调度控制器。内存子系统，cpu子系统等等
- 层级（hierarchy）：一个层级是一个树状的cgroup结构。在一个层级内，子节点默认继承父节点所挂载的子系统。一个系统中可以有多个层级.

实现特点
- 将一系列系统任务（及其子任务）整合到不同组内，以组为单位进行资源分配和管理
- 通过伪文件系统的方式实现，通过mount指令能看到挂载信息。程序可以通过文件操作实现cgroups操作。在/proc/cgroups中就能看到当前内核所有的cgroups子系统。
- 管理粒度以线程为单位，提供资源限制、优先级分配、资源统计、任务控制四大功能

规则
- 一个层级可以绑定多个子系统。但反之则有限制，一个子系统绑定到多个层级时，每个层级只能有该子系统。
- 一个任务可以加入多个层级的不同控制组，但是在一个层级内，只能加入一个控制组。
- 子进程fork/clonse后会在父进程所在的控制组内，但是随后可以对所自身在控制组进行移动。

原理
```c
struct task_struct {
#ifdef CONFIG_CGROUPS
	/* css_set共享指针（只有所属cgroup完全相同的进程才会共享） */
	struct css_set *cgroups;
	/* 存储使用同一个css_set的链表 */
	struct list_head cg_list;
#endif
};

struct css_set {
  /* 引用计数 */
	atomic_t refcount;
  /* 用于存储css_set的散列表结构 */
	struct hlist_node hlist;
  /* 存储当前cgroup的全部进程的链表 */
	struct list_head tasks;
  /* 指向cg_cgroup_link结构的链表 */
	struct list_head cg_links;
  /* 用于获取绑定的各个子系统 */
	struct cgroup_subsys_state *subsys[CGROUP_SUBSYS_COUNT];
  /* 用于内存read-copy-update回收机制 */
	struct rcu_head rcu_head;
};

/* 实际上不同子系统在实现时，该结构体只是其成员之一，这里存储的是子系统的通用内容 */
struct cgroup_subsys_state {
  /* 指向子系统所附加到的cgroup */
  struct cgroup *cgroup;
  /* 引用计数 */
  atomic_t refcnt;
  unsigned long flags;
  struct css_id *id;
};

struct cgroup {
  unsigned long flags;
  atomic_t count;
  /* 兄弟节点链表，链接到兄弟节点sibling和父节点的children */
  struct list_head sibling;
  /* 子节点链表，链接到子节点的sibling */
  struct list_head children;
  /* 父节点 */
  struct cgroup *parent;
  /* 当前cgroup在虚拟文件系统中的挂载目录 */
  struct dentry *dentry;
  /* 绑定的子系统 */
  struct cgroup_subsys_state *subsys[CGROUP_SUBSYS_COUNT];
  /* 所属层级 */
  struct cgroupfs_root *root;
  /* 当前所属层级的根cgroup */
  struct cgroup *top_cgroup;
  /* 指向cg_cgroup_link结构的链表 */
  struct list_head css_sets;
  struct list_head release_list;
  struct list_head pidlists;
  struct mutex pidlist_mutex;
  struct rcu_head rcu_head;
  struct list_head event_list;
  spinlock_t event_list_lock;
};

/* 该结构完成css_set和cgroup的多对多双向绑定 */
struct cg_cgroup_link {
  /* 是cgroup->css_sets所指向的链表 */
  struct list_head cgrp_link_list;
  /* 所属的cgroup */
  struct cgroup *cgrp;
  /* 是css_set->cg_links所指向的链表 */
  struct list_head cg_link_list;
  /* 所属的的css */
  struct css_set *cg;
};

struct cgroupfs_root {
  /* 该层级的文件超级块 */
  struct super_block *sb;
  /* 计划将要绑定到当前层级的子系统 */
  unsigned long subsys_bits;
  /* 该层级唯一的id */
  int hierarchy_id;
  /* 已绑定的子系统 */
  unsigned long actual_subsys_bits;
  struct list_head subsys_list;
  /* 该层级的根cgroup */
  struct cgroup top_cgroup;
  /* 该层级cgroup的个数 */
  int number_of_cgroups;
  /* 将所有cgroupfs_root链接的链表 */
  struct list_head root_list;
  unsigned long flags;
  char release_agent_path[PATH_MAX];
  char name[MAX_CGROUP_ROOT_NAMELEN];
};

/* 定义子系统的基础操作 */
/* 和cgroup_subsys_state一样，该结构只是子系统所需实现的通用部分，会被各自实现包含在内作为成员 */
struct cgroup_subsys {
  struct cgroup_subsys_state *(*create)(struct cgroup_subsys *ss,
                        struct cgroup *cgrp);
  int (*pre_destroy)(struct cgroup_subsys *ss, struct cgroup *cgrp);
  void (*destroy)(struct cgroup_subsys *ss, struct cgroup *cgrp);
  int (*can_attach)(struct cgroup_subsys *ss, struct cgroup *cgrp,
            struct task_struct *tsk, bool threadgroup);
  void (*cancel_attach)(struct cgroup_subsys *ss, struct cgroup *cgrp,
            struct task_struct *tsk, bool threadgroup);
  void (*attach)(struct cgroup_subsys *ss, struct cgroup *cgrp,
          struct cgroup *old_cgrp, struct task_struct *tsk,
          bool threadgroup);
  void (*fork)(struct cgroup_subsys *ss, struct task_struct *task);
  void (*exit)(struct cgroup_subsys *ss, struct task_struct *task);
  int (*populate)(struct cgroup_subsys *ss,
          struct cgroup *cgrp);
  void (*post_clone)(struct cgroup_subsys *ss, struct cgroup *cgrp);
  void (*bind)(struct cgroup_subsys *ss, struct cgroup *root);
  int subsys_id;
  int active;
  int disabled;
  int early_init;
  bool use_id;
#define MAX_CGROUP_TYPE_NAMELEN 32
  const char *name;
  struct mutex hierarchy_mutex;
  struct lock_class_key subsys_key;
  struct cgroupfs_root *root;
  struct list_head sibling;
  struct idr idr;
  spinlock_t id_lock;
  struct module *module;
};
```
- 在linux的进程结构体task_struct中，添加了当前所属的控制组的指针。由于进程可以属于多个cgroup，反之一个cgroup也可以包含多个进程，因此引入了css_set完成多对多的映射
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

## Docker提供的隔离
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
  2. docker host上的docker daemon接收到http请求， 由http2p agent拉取docker镜像到本地
  3. dockerd挂载容器实例根路径rootPath、配置bundle目录
  4. dockerd通过grpc调用containerd创建容器实例
  5. containerd执行runc
  6. runc读取bundle下配置，以挂载路径rootPath作为rootfs启动容器实例
- 监控：eBPF（KProbe） based metrics，非侵入式。（选择要探测的内核函数，eBPF调用kProbe，写入map，用户态读取map，将进程级数据聚合到cgroup级别）

## 指令示例
1. docker
  ```bash
  # 创建使用bash的可交互容器，命名为redis-slave1，（通过在host中定义master）连接到名为redis-master的容器
  docker run -it --name redis-slave1 --link redis-master:master redis /bin/bash

  # 查看容器运行时配置（常看volume、ip等）
  docker inspect XXXX

  # 查看容器运行时磁盘使用
  docker system df -u

  # 创建redis容器，并映射主机路径A到容器路径B（A不存在则默认创建）
  docker run -v /A:/B redis

  ```
2. docker-compose

## 参考
《Docker容器与容器云》

[Linux Cgroups详解(一)](https://www.cnblogs.com/lisperl/archive/2012/04/17/2453838.html)、[Linux Cgroups详解(二)](https://www.cnblogs.com/lisperl/archive/2012/04/18/2455027.html)
