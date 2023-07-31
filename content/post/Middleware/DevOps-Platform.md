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
本篇讲述DevOps相关内容中，和运维平台相关的内容。本文内容侧重应用，会记录一些实用命令、参考网站。
<!--more-->
## 运维历史
## 常见中间件
### OpenStack
### Docker
![Docker命令逻辑关系](/images/middleware/docker-cmd-routine.jpg)
<center>Docker命令逻辑关系</center>

1. 安装：这里推荐[博客](https://yeasy.gitbook.io/docker_practice/install)。推荐使用Docker官方提供的脚本完成自动安装，另外还要记得为docker配置用户用户组，如下
    ```bash
    # 使用mirror参数来指定镜像站
    sudo sh get-docker.sh --mirror Aliyun

    # 创建用户组docker（安装后其实已经存在）
    sudo groupadd docker
    # 将当前用户添加到docker组
    sudo usermod -aG docker $USER
    ```
    > 注意：如果实用官方脚本直接安装，则安装后需要自行配置一下镜像源（/etc/docker/daemon.json）。安装时的--mirror只是针对于安装，并不是docker使用时的registry镜像源

    > 如果仍发生权限问题，可以检查docker.sock权限是否有误，其属组应该是docker，可以修改/lib/systemd/system/docker.socket文件并重启，[参考](https://www.codeprj.com/blog/588a231.html)
2. 编写Dockerfile，打包image镜像
    ```dockerfile
    # Dockerfile

    # 前往docker hub查看所需的各类依赖的用法
    # alpine版本是一个略微精简的node环境，进一步可以去了解Alpine Linux
    FROM node:alpine
    # 将当前文件夹拷贝到容器的/my-app位置
    COPY . /my-app
    # 设置工作目录
    WORKDIR /my-app
    # 容器内计划执行的程序命令
    CMD node main.js
    ```
    ```js
    // main.js
    console.log('hello world')
    ```
    ```bash
    # 命令行内
    # image命名为hello-world，打包当前目录
    docker build -t hello-world .
    # 执行
    docker run hello-world
    # 查看容器执行情况
    docker ps -a
    # 进入容器(xxxx为container id)
    docker exec -it xxxx bash
    ```
3. 注意项：
   - Docker的版本发生过一次变更。从原先的docker（docker-engine，版本号不大于1.13.1），分化为docker-ce、docker-ee，社区版和企业版。在不同系统上安装时，需要注意优先删除旧版的docker。
   - docker-compose是基于docker的一个子项目，提供更方便的容器编排，即如果有一个项目，需要启动多个容器，则可以使用docker-compose，编写docker-composer.yml文件，达成敲一次命令，启动所有的容器。
4. 推荐视频：
   1. [Docker Containers and Kubernetes Fundamentals – Full Hands-On Course](https://www.youtube.com/watch?v=kTp5xUtcalw)
   2. [Docker Tutorial for Beginners](https://www.youtube.com/watch?v=pTFZFxd4hOI)
### Kubemetes
k8s简单介绍：支持灵活的调度机制、丰富的组件元语（支持无状态、有状态服务的组件元语，方便扩缩容）、支持高性能的服务注册/订阅功能、扩展性强支持插件化二次开发。

service port：集群内部应用访问方式。node port：集群外部服务访问方式。k8s通过改写相关iptable支持请求转发到相关pod。

调度机制：亲和性和反亲和性（出于容灾考虑）。【基于pod的label进行匹配】

## 总结
&emsp;&emsp;软件开发行业需要解决的若干问题总而言之就是：提高代码和组件分发、开发、合并、测试、部署上线、回滚的效率。

## 参考
1. [Linux核心调度器之周期性调度器scheduler_tick--Linux进程的管理与调度(十八） ](https://www.cnblogs.com/linhaostudy/p/9867364.html)
2. [Linux系统核心调度器——主调度器schedule函数详解](https://blog.csdn.net/weixin_42092278/article/details/88778435?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)
3. [gitbook《Docker 从入门到实践》](https://yeasy.gitbook.io/docker_practice/)
4. [Linux基金 指导手册Linux Foundation Referenced Specifications](https://refspecs.linuxfoundation.org/)