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
## 数据库基本能力
```c
// 持久化参数
struct saveparam {
    // 秒数
    time_t seconds;
    // 修改数
    int changes;
}
// 数据库
struct redisDb {
    // 数据库键空间，存储所有键值对儿
    dict *dict;
    // 过期时间
    dict *expires;
}
// 服务器数据库
struct redisServer {
    // 保存服务器中的所有数据库
    redisDb *db;
    // 服务器数据库数量
    int dbnum;

    // 持久化条件
    struct saveparam *saveparams;
    // 修改计数器
    long long dirty;
    // 上一次持久化时间戳
    time_t lastsave;
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
    - 数据库键空间的键就是用户存储值到数据库时用的键，每一个键就是一个字符串对象
    - 键空间的值就是用户存储在数据库中的值，可以是字符串对象、列表对象等等
    - 大部分命令都会作用于键空间（或者经过键空间）：添加、删除、更新、取值
    - 键空间的维护操作：
        - 访存键均会进行命中（hit）和不命中（miss）的累计计算
        - 成功读取一个键回更新键的LRU（最后一次使用）时间，可用于计算键的空闲时间
        - 如果访问时发现键过期，则会直接删除
        - 如果有客户端WATCH一个键，则该键更新后会被标记为脏，让后续事务程序注意到修改
        - 键更新后会将dirty计数增1，用以触发服务器的持久化和复制操作
        - 如果有通知功能，键更新后还会进行通知发送
1. 键的生存周期：
    - 生存时间：Time To Live（TTL），在经过该时间后，服务器会删除生存时间归0的键。
    - 过期时间：Expire Time，在到达该时间点时，服务器删除该键。时间戳使用UNIX时间戳。
    - 过期时间的保存：实际上所有的生存时间、过期时间最终都可以用一个过期时间戳（毫秒）进行表示。即字典expires。过期字典的键和键空间的键会共享同一个键对象。
    - 清除过期时间：服务器查找过期字典中的指定键，移除键和值之间在过期字典中的关联。
1. 过期键的删除策略：
    - 主动删除策略
        - 定时删除：设置过期时间的同时设置定时器，准时删除。对CPU不友好。且Redis中的时间事件由无序链表实现，查找事件的复杂度为$O(N)$。
        - 定期删除：定期检车数据库中的键并进行删除。对CPU、内存性能进行平衡的方案。但也需要合理配置删除频率和删除策略。算法将会定期，随机选择一定数量的键，检查并清理。
    - 被动删除策略
        - 惰性删除：下次访问时检查并删除。对内存不友好。
1. 过期键和持久化
    - 创建新RDB文件时，所有过期键均不会被保存。
    - 加载RDB文件时，主服务器会直接删除过期键，从服务器不会（但随后主从同步还是会被删除）
    - AOF持久化模式下，只有当过期键确实被删除时，才会进行DEL命令的记录。AOF重写也不会记录和过期键有关的操作。
    - 复制模式下，**主服务器控制**从服务器上过期键的删除，而不是连接到从服务器的客户端（即使接收到命令，也不会去删除）。未删除时，该过期键仍然能够被查询返回。

## 订阅和通知
1. 键空间通知（key space notification）
    1. 订阅指定的键的所有操作命令
1. 键事件通知（key event notification）
    1. 订阅指定的命令、事件，如DELETE命令
1. 实现：
    1. 功能入口：
    ```c
    /* 核心函数
    * type  当前发送的通知类型(服务器可能禁止某些通知发送)
    * event 事件名称
    * key   键
    * dbid  当前数据库号
    *
    */
    void notifyKeyspaceEvent(int type, char *event, robj *key, int dbid);
    ```
    - 调用：每一个命令实现时，都会在执行后判断是否成功，并调用该函数，进行通知的发送，例如
    ```c
    void saddCommand(redisClient *c) {
        // SADD命令、集合添加元素成功后
        notifyKeyspaceEvent(REDIS_NOTIFY_SET, "sadd", c->argv[1], c->db->id);
    }
    ```
    - 发送：publishMessage函数，该函数也是PUBLISH命令的底层实现。

## RDB持久化
1. RDB文件是一个经过压缩的二进制文件。
1. 创建：
    - SAVE：阻塞服务器进程直到RDB文件创建完毕，阻塞期间不处理任何请求。
    - BGSAVE：派生子进程创建RDB文件，父进程继续处理请求。但此时服务器拒绝执行SAVE、BGSAVE，并延迟执行BGREWRITEAOF。
1. 加载：启动时自动检测RDB文件并加载（阻塞加载）
    - 由于AOF频率比RDB高，因此当开启AOF持久化时，Redis将优先使用AOF文件加载，而不使用RDB文件
1. 自动持久化
    1. 配置：可以配置触发BGSAVE的时间窗口A和修改次数B，代表在A时间段内，至少修改B次后触发BGSAVE。
    1. 检查：服务器定时任务函数serverCron默认100毫秒执行一次，其中就包括对持久化的评估。
    1. 其他细节：
        1. dirty计数器：统计自上次持久化后修改的元素数，而非命令数量。如一次性添加多个键值对儿，增加dirty多次，而非一次。

## 数据库命令
1. SELECT X：选择第X个数据库
1. EXPIRE KEY SEC：设置生存时间（秒）
1. PEXPIRE KEY MSEC：设置生存时间（毫秒）
1. EXPIREAT KEY TIMESTAMP：设置过期时间戳（秒）
1. PEXPIREAT KEY TIMESTAMP：设置过期时间戳（毫秒）
1. TTL/PTTL KEY：查询一个键距离被删还有多长时间
1. PERSIST KEY：清除过期时间
1. FLUSHDB：强制清除数据库全部键值