---
title: "设计模式-同步异步"
date: 2022-04-02T15:02:03+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
在日常使用的各类操作系统级接口（系统调用）中，时常有同步、异步，阻塞、非阻塞的区分。本文将结合实际对其进行总结。应用程序级接口也可以由此类推。
<!--more-->
## 基本概念
1. IO步骤：一般分为内核等待数据的阶段，以及数据从内核复制到用户空间这两个部分。
1. 本文针对的接口是系统接口，即系统调用
1. 同步和异步：对用户进程和系统内核的交互过程的区分
    - 同步：用户进程触发IO操作，并等待或者轮询去查看IO操作是否就绪。调用返回时获得有效结果。
    - 异步：用户进程触发IO操作后，去处理其他事项，并由系统通知IO操作就绪。调用返回不代表操作结束。
1. 阻塞和非阻塞：针对用户进程访问数据时，根据IO操作的就绪状态采取的进程调度应对方式的区分
    - 阻塞：IO操作未完成，则等待完成，进程挂起
    - 非阻塞：即使IO操作未完成，也立即返回
## IO模型
1. 阻塞IO（Blocking IO）：发起IO后一直阻塞直到完全完成。
1. 非阻塞IO（Unblocking IO）：发起IO后一直检查内核数据是否准备就绪，就绪后开始从内核向用户空间复制（复制阶段仍然阻塞）。
    - 反复检查阶段耗费大量CPU资源
1. IO复用（IO multiplexing）：select、poll、epoll，和非阻塞类似，但可以同时对多个IO操作进行检查，获得一个返回后，针对该IO操作调用相应的实际处理函数。检查IO操作时，会阻塞到有操作返回结果（或者超时）
1. 信号驱动IO（signal driven IO，SIGIO）：注册信号处理函数，在对应数据在内核中准备好时，通知处理函数进行处理，处理阶段仍然阻塞。
1. 异步IO（asynchronous IO）：真正高效的接口。发起IO操作后，直到数据已经在用户空间准备完成，才会调用对应的处理程序。

## Linux、Windows异步
- Linux：aio
- Linux：io_uring
- Windows：IOCP

## Reactor和Proactor

## 参考：
- [5种网络IO模型](https://zhuanlan.zhihu.com/p/54580385)
- [Linux内核全新异步IO引擎(一)](https://zhuanlan.zhihu.com/p/334658432)
- [Linux内核全新异步IO引擎(二)](https://zhuanlan.zhihu.com/p/334763504)