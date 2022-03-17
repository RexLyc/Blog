---
title: "Redis：数据结构和用法"
date: 2022-02-18T20:16:44+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/redis.jpg
math: true
---
redis提供广泛的数据结构种类，用于在不同业务场景中使用。
<!--more-->
# 底层
1. 字符串
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
1. 链表
    ```c
    // 链表节点
    typedef struct listNode {
        struct listNode *prev;
        struct listNode *next;
        void *value;
    };
    // 链表
    typedef struct list {
        listNode *head;
        listNode *tail;
        unsigned long len;
        // 复制用函数指针
        void *(*dup) (void *ptr);
        // 释放用函数指针
        void (*free) (void *ptr);
        // 节点比较函数
        int (*match) (void *ptr, void *key);
    }
    // 大量api
    // listSetDupMethod / listLength / listPrevNode / listDup ...
    ```
    1. 用于列表键（元素较多时）、发布订阅、慢查询、监视器登功能的实现。
1. 字典
    ```c
    // 字典项
    typedef struct dictEntry {
        // 键
        void *key;
        // 值
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v;
        // 指向下一个字典项，链接法解决冲突
        struct dictEntry *next;
    }

    // 字典表
    typedef struct dictht {
        dictEntry **table;
        // 总是2^n
        unsigned long size;
        // 总是size-1
        unsigned long sizemask;
        unsigned long used;
    }
    
    // 类型相关函数结构体
    typedef struct dictType {
        // 散列函数指针
        unsigned int (*hashFunction) (const void *key);
        // 键复制函数指针
        void *(*keyDup) (void *privdata, const void *key);
        // 值复制函数指针
        void *(*valDup) (void *privdata, const void *obj);
        // 键对比函数指针
        int (*keyCompare) (void *privdata, const void *key1, const void *key2);
        // 键销毁函数指针
        void (*keyDestructor) (void *privdata, void *key);
        // 值销毁函数指针
        void (*valDestructor) (void *privdata, void *obj);
    }

    // 字典
    typedef struct dict {
        // 字典相关函数
        dictType *type;
        // 个性化数据
        void *privdata;
        // 字典表数组
        dictht ht[2];
        // rehash标记，未rehash时为-1
        int rehashidx;
    }
    ```
    1. 也被称为符号表（symbol table）、关联数组（associative array）、映射（map）。数据库、哈希键的底层实现之一。
    1. 散列值计算：index= dict->type->hashFunction(key) & dict->ht\[x\].sizemask;
    1. 一般的使用[MurmurHash2](http://www.360doc.cn/article/4238731_210314514.html)算法作为散列函数。
    1. 键冲突时，使用链表法，将新值添加到当前键链表的最前端。
    1. 键冲突时，使用链表法，将新值添加到当前键链表的最前端。1. 大部分时候数据存储于ht[0]。rehash时会在ht[1]开辟更大空间，完成后将ht[0]和ht[1]交换。
    1. rehash
        1. 扩展操作：ht[1]大小定为首个不小于ht[0].used*2的$2^n$
        1. 收缩操作：ht[1]大小定为首个不小于ht[0].used的$2^n$
        1. rehash条件：未BGSAVE、BGREWRITEAOF，且负载因子大于1；或者正在BGSAVE、BGREWRITEAOF，但负载因子大于5；负载因子小于0.1，进行收缩。
        1. rehash过程并不是立刻全部，而是渐进式的，每次访问到仍在ht[0]中的剩余键，进行迁移。
        1. rehashidx代表当前已经迁移的键的个数。rehash开始后，每次CRUD操作，都会额外进行对ht[0].table[rehashidx]的迁移。并自增rehashidx。
1. 跳跃表
    ```c
    // 跳跃表节点
    typedef struct zskiplistNode {
        // 当前节点各层
        struct zskiplistLevel {
            // 当前层后继
            struct zskiplistNode *forward;
            // 当前层后继的距离
            unsigned int span;
        } level [];
        // 当前节点前驱
        struct zskiplistNode *backward;
        // 当前节点分数（各节点升序排列，可重复）
        double score;
        // 保存的对象，跳跃表内唯一
        robj *obj;
    } zskiplistNode;

    // 跳跃表
    typedef struct zskiplist {
        // 表头和表尾
        struct zskiplistNode *header, *tail;
        // 节点总数（不含表头）
        unsigned long length;
        // 最大层数（不含表头）
        int level;
    }
    ```
    1. 有序集合键的底层实现之一。也用于集群模式节点中做内部数据结构。
    1. 层数以幂次定律生成，越大概率越低，取值范围[1,32]。表头节点是特殊的，不存储数据，但有全部32层。
    1. 遍历过程中累计span，可以得出目标值在跳跃表中的排位。 
    1. 排序先按score，score相同，则按保存的对象*obj排序。
1. 整数集合
    ```c
    typedef struct intset {
        // 编码方式
        uint32_t encoding;
        // 集合元素总数
        uint32_t length;
        // 保存元素的数组
        int8_t contents[];
    }
    ```
    1. 集合键的底层实现之一。针对数量较少的整数存储。
    1. 元素升序排列，不允许重复。
    1. encoding取值有：INTSET_ENC_INT16、INTSET_ENC_INT32、INTSET_ENC_INT64等。
    1. 升级：当新元素长度超过现有encoding能表达的范围时，整个数组将会进行升级，原有元素也会使用表达范围更大的编码方式进行存储。步骤包括：扩展数组空间，转存已有元素，添加新元素（一定在数组头或尾）。
    1. 不支持降级。
1. 压缩列表
    ```c
    
    ```
    1. 列表键和哈希键的底层实现之一。主要针对数量少，长度短/数值小的关键字。
    1. 
# 键值对
1. redis数据库中最基础的数据结构。key-value pair。
1. 键总是字符串对象。而值则可以是：字符串、列表（list）、哈希（hash）、集合（set）、有序集合（zset）。
    
# redis-cli
1. redis的命令行控制工具。
1. 实用技巧
```bash
# 通过管道方式，批量执行控制命令
cat insert.txt | redis-cli --pipe
```
已阅读至第三章