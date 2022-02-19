---
title: "Redis：开坑篇"
date: 2022-02-18T20:16:22+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/redis.jpg
---
redis是一种被普遍使用的的内存数据库。本系列结合原理和使用方式进行讲解。
<!--more-->
# 概述
1. 截至2022年2月，redis是一个BSD许可下（允许商用，只需要附上原许可证）的内存数据库。
1. redis的设计目的就是性能至上。常被用做数据库，缓存，消息存储（message broker）。
1. 提供超多种类的数据结构。
1. 支持事务、不同的缓存策略、持久化、集群化等功能。
1. 采用[ANSI_C](https://zh.wikipedia.org/wiki/ANSI_C)编写。
1. 由于基于《Redis设计与实现》一书（Redis2.9）编写本博客，因此需要特别注意浏览新特性一章。
# 系列分章
1. 数据结构和用法
1. 基本原理
1. 集群用法
1. 其他
1. 新特性
# 参考文献
1. 《Redis设计与实现》2014年版
1. [redis官方文档](https://redis.io/documentation)