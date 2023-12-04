---
title: "《Netty In Action》读书笔记"
date: 2023-12-04T22:14:02+08:00
categories:
- 读书笔记
- 技术书
tags:
- 施工中
- Netty
- 框架
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/netty.png
draft: true

---
Netty作为一个广泛应用的Java高性能网络框架，不仅可以作为框架使用，其本身的设计模式和思路也很值得学习。
<!--more-->
## 基本架构
1. 基本术语：
   1. Channel：在Netty中代表了到一个通信实体的连接，可以对连接进行打开、关闭，并对活跃连接上进行读写，
   2. 回调：Netty使用回调来处理各类事件，使用回调时，须泛化实现对应接口的事件回调函数
   3. Future：Netty提供了自己的实现```ChannelFuture```，可以通过对其添加监听器```ChannelFutureListener```，作为另一种在事件结束时通知应用程序的方式
   4. 事件和ChannelHandler：Netty提供的事件包括但不限于
        - 连接开关
        - 记录日志
        - 数据转换
        - 流控制
        - 用户事件
        </br>这些事件将会根据其数据发送方向，分别在ChannelHandler中由预设、或用户实现的各种事件处理器进行链式处理。
1. 基本原理：
    - Netty基于NIO，整体原理就是由Selector监听I/O事件，并派发给对应的处理线程
    - Netty为每一个Channel创建一个事件循环EventLoop
    - EventLoop在执行期间注册回调，并将事件派发

## 核心架构
### BossGroup

### WorkerGroup


## 线程模型

## 编解码器

## 网络协议

## 案例分析

## 参考资料
1. 《Netty In Action》