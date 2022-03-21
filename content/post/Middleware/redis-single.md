---
title: "Redis：单机数据库"
date: 2022-03-18T22:17:17+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/redis.jpg
---
单机数据库是Redis的基础形式。本章讲解相关概念和原理。
<!--more-->
# 数据库
```c
// 服务器数据库
struct redisServer {
    // 保存服务器中的所有数据库
    redisDb *db;
    // 服务器数据库数量
    int dbnum;
}
// 客户端数据库
struct redisClient {
    // 当前正使用的数据库
    redisDb *db;
}
```
1. 创建：
    - 根据服务器配置，创建若干数据库，默认是16个。
1. 键空间：
    - 

# 数据库命令
1. SELECT X：选择第X个数据库
1. 