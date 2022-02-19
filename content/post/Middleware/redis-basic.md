---
title: "Redis：数据结构和用法"
date: 2022-02-18T20:16:44+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/redis.jpg
---
redis提供广泛的数据结构种类，用于在不同业务场景中使用。
<!--more-->
# 键值对
1. redis数据库中最基础的数据结构。key-value pair。
1. 键总是字符串对象。而值则可以是：字符串、列表（list）、哈希（hash）、集合（set）、有序集合（zset）。
1. 底层：
    ```c
        struct sdshdr {
            // 所存储的字符串长度
            int len;
            // 空闲字节数
            int free;
            // 字节数组，用于保存
            char buf[];
        }
        // 更多API
        // sdsnew / sdsempty / sdsfree / sdslen / sdsavail / sdsdup ...
    ```
    1. redis的实现中，大部分地方都使用其自身实现的简单动态字符串（simple dynamic string，SDS），而非使用传统C字符串。作为键值的字符串其存储方式就是SDS。
    1. buf中仍然必须保留结尾的\0，因此len+free+1才是buf的实际的总字节数。这使得部分C库函数仍然有效。
    1. 分配buf时往往大于实际字符串长度。SDS整体上都贯彻了以空间换时间的思路。
    1. 二进制安全，即允许内部有\0。
    1. SDS还作为缓冲区，用于进行AOF持久化等工作。
    1. 优化策略：
        - 空间预分配：free=min(1MB,len)。
        - 惰性空间释放：字符串缩短时，并不立刻释放buf。
# redis-cli
1. redis的命令行控制工具。
1. 实用技巧
```bash
# 通过管道方式，批量执行控制命令
cat insert.txt | redis-cli --pipe
```
已阅读至第三章