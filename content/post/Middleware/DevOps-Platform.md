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
1. Docker
1. OpenStack
1. Kubemetes
    - k8s简单介绍：支持灵活的调度机制、丰富的组件元语（支持无状态、有状态服务的组件元语，方便扩缩容）、支持高性能的服务注册/订阅功能、扩展性强支持插件化二次开发。
    - service port：集群内部应用访问方式。node port：集群外部服务访问方式。k8s通过改写相关iptable支持请求转发到相关pod。
    - 调度机制：亲和性和反亲和性（出于容灾考虑）。【基于pod的label进行匹配】

## 参考