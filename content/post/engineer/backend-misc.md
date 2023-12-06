---
title: "工程技术：后端合集"
date: 2023-12-06T13:14:26+08:00
categories:
- 
tags:
- 
# thumbnailImagePosition: left
# thumbnailImage: //example.com/image.jpg
draft: true
---
本文记录一些在后端开发中常见的一些工程技术，用来提高稳定性，提高性能等。可能是一些设计思路或实现范式。
<!--more-->
## 流量限制算法
1. 漏桶：以绝对固定的速率接受请求并进行处理，像是一个桶以固定的速率漏水一样
1. 令牌桶：以固定的速率向桶内放令牌，拿到令牌的请求可以进行处理，因此能够接受一定量的大并发
1. 参考：
    - [高并发系统限流-漏桶算法和令牌桶算法](https://www.cnblogs.com/xuwc/p/9123078.html)