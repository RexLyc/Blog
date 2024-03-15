---
title: "边学边用linux-内核篇"
date: 2024-03-15T11:14:25+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 命令行
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
draft: true
---
本文记录一些Linux内核原理，内核数据结构，内核算法，内核参数相关的知识
<!--more-->
> 本章节内容也可能会在本系列的其他文章中出现。主要是为了方便集中查阅。

## 内核参数
一些内核参数是可以配置的，可以通过如下指令进行查看
```bash
# 查看内核参数
sysctl -A 
```

一些比较重要的参数
| 参数名 | 内容 | 参数值（举例） | 含义 |
| --- | --- | --- | --- |
| net.ipv4.tcp_rmem、net.ipv4.tcp_wmem | tcp接收、发送缓冲区大小 | 4096 87300 6291456 | 最小4k、默认87k、最大6M |

## 内核数据结构

## 内核算法

## 内核原理

## 参考
1. [知乎-陈硕：Linux 中每个 TCP 连接最少占用多少内存？](https://zhuanlan.zhihu.com/p/25241630)