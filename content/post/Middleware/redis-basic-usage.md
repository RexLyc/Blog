---
title: "Redis：基本使用篇"
date: 2022-01-28T17:12:56+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/redis.jpg
---
Redis是优秀的内存键值数据库，本文简述Redis的基本原理和使用方式。
<!--more-->
# 历史
# 基本原理
# 数据结构
# 最佳实践
# redis-cli
1. redis-cli提供对redis的控制和数据操作
1. 基本使用
```bash
cat insert.txt | redis-cli --pipe # 将写有操练指令的文件管道传输给redis服务器
```
# 参考