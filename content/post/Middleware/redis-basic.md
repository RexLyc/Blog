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
# 键值对
1. redis数据库中最基础的数据结构。key-value pair。
1. 键总是字符串对象。而值则可以是：字符串、列表（list）、哈希（hash）、集合（set）、有序集合（zset）。
1. 因为键总是字符串，所以称呼“字符串键”，指值是字符串，同理其他键。

# 底层
1. 字符串：raw
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
1. 双端链表：linkedlist
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
1. 字典：hashtable
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
1. 跳跃表：skiplist
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
1. 整数集合：intset
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
1. 压缩列表：ziplist
    1. 字段组成
        1. 压缩列表
        | 字段名称 | 类型 | 长度（字节） | 作用 |
        | --- | --- | --- | --- |
        | zlbytes | uint32_t | 4 | 记录压缩列表总的内存字节数 |
        | zltail | uint32_t | 4 | 记录表尾节点距离起始地址的字节数 |
        | zllen | uint16_t | 2 | 记录了压缩列表包含的节点数量 |
        | entryX | 列表节点 | 不定 | 压缩列表包含的各个节点 |
        | zlend | uint8_t | 1 | 0xFF标记 |
        1. 压缩列表节点
        | 字段名称 | 长度 | 作用 |
        | --- | --- | --- |
        | previous_entry_length | 1或5字节 | 记录前一个节点的长度，如果大于等于0xFE，则以5字节存储，并以0xFE标记开头 |
        | encoding | 1、2、5字节 | 最高两个bit位00、01、10则代表是字节数组，其后数值代表数组长度；最高两个bit位11则代表整数编码，其后长度代表整数值类型和长度 |
        | content |  |  |
    1. 列表键和哈希键的底层实现之一。主要针对数量少，长度短/数值小的关键字。
    1. 由于zllen的类型限制，当压缩列表数量大于等于UINT16_MAX时，真正的节点数量需要遍历整个列表才能得知。
    1. 每个节点可以保存一个字节数组或者一个整数值
        - 字节数组：长度<=63字节、<=16383、<=4294967295
        - 整数值：4bit（0~12）、1字节有符号整数、3字节有符号整数、int16_t类型整数、int32_t类型整数、int64_t类型整数
    1. 编码较为复杂，可以参考[ziplist](https://zhuanlan.zhihu.com/p/144211926?from_voters_page=true)。
    1. 连锁更新：由于每个节点存储前一个节点的长度，所以当前驱大小发生变更时（插入、删除），有可能会导致多个节点连锁更新。但平均情况下，连锁更新对性能的损耗并不是很高。
    1. 压缩列表的核心目的是节约内存。
# 对象
```c
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 底层数据结构指针
    void *ptr;
    // 引用计数
    int refcount;
    // 空转时长
    unsigned lru:22;
}
```
1. 概述：Redis实现了一套对象系统。并在系统内使用前面提到的各类基础数据结构。Redis实现了基于引用计数技术的内存回收机制。并基于引用计数进行对象共享。Redis的对象带有访问时间记录信息，可以用于计算键的空转时长。可以配置优先删除这些键。
1. 类型指存储的数据类型，包括：REDIS_STRING / REDIS_LIST / REDIS_HASH / REDIS_SET / REDIS_ZSET等
1. 编码，指存储数据所使用的底层实现，包括：</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_INT、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_EMBSTR、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_RAW、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_HT、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_LINKREDIST、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_ZIPLIST、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_INTSET、</br>
    &emsp;&emsp;&emsp;REDIS_ENCODING_SKIPLIST
    - 有些类型拥有多个编码方式
    - raw即SDS，embstr其实也是SDS。但raw在分配时会分别调用robj、SDS的内存分配函数，而embstr只需要调用一次内存分配，直接分配一个同时包含robj、sds的连续内存区域。释放同理。
1. 对象类型：
    1. 字符串对象
        1. 编码：
            - int：如果字符串保存整数值，且在long范围内，则void*转为long
            - embstr：小于等于32字节的字符串
            - raw：大于32字节的字符串
        1. 编码转换：当执行了一些指令，使得对象保存的数据类型发生变化，则将会进行转换。如对int类型连接字符串，则会转为raw。对embstr的任何修改，都会转为raw。
        1. 字符串对象会被嵌套在其他类型对象内部。
        1. 部分字符串键命令：SET、GET、APPEND、INCRBYFLOAT、INCRBY、DECRBY、STRLEN、SETRANGE、GETRANGE
            - embstr、raw不支持INCRBY、DECRBY
    1. 列表对象
        1. 编码：
            - ziplist：列表对象保存的所有字符串对象的长度都小于64字节，且总对象数量少于512。阈值可配置。
            - linkedlist：其他情况
        1. 编码转换：当编码条件不满足时进行转换。
        1. 部分列表键命令：LPUSH、RPUSH、LPOP、RPOP、LINDEX、LLEN、LINSERT、LREM（指定删除）、LTRIM（指定范围外删除）、LSET
    1. 哈希对象
        1. 编码：
            - ziplist：所有键值对中键、值的字符串对象大小都小于64字节，且键值对总数少于512个。阈值可配置。
            - hashtable：其他情况
        1. ziplist的存储方式：按照键-值-键-值的顺序，依次添加压缩列表节点到列表尾部。
        1. 编码转换：编码条件不满足时进行转换。
        2. 部分哈希键命令：HSET、HGET、HEXISTS、HDEL、HLEN、HGETALL
    1. 集合对象
        1. 编码：
            - intset：所有元素都是整数值，且总数不超过512个。阈值可配置。
            - hashtable：其他情况。字典中的值字段都设为NULL。
        1. 编码转换：不满足时转换。
        1. 部分集合键命令：SADD、SCARD（获取元素总数）、SISMEMBER、SMEMBERS、SRANDMEMBER（随即返回）、SPOP（随机弹）、SREM（删除指定）
    1. 有序集合对象
        1. 编码：
            - ziplist：保存元素的对象长度都小于64字节，且元素总数小于128个。阈值可配置。
            - skiplist：其他情况
        1. 编码实现：
            - ziplist：按照值（字符串对象）-分值（score）-值-分值保存，以分值升序排列。
            - skiplist：此处的skiplist并非底层的跳跃表实现，而是同时包含字典和跳跃表的对象，且同一个值及其分值以指针共享。
                ```c
                typedef struct zset {
                    // 保存实际存储对象
                    zskiplist *zsl;
                    // 保存对象及其在zskiplist中的分值
                    dict *dict;
                }
                ```
                > 同时存储字典和跳跃表是为了能同时提供高效率的单点访存和范围访存。
        1. 编码转换：不满足时转换。
        1. 部分有序集合键命令：ZADD、ZCARD、ZCOUNT（统计指定分值范围内的元素数量）、ZRANGE、ZERVRANGE（反向返回指定索引范围内元素）、ZRANK、ZREVRANK（反向分值排名）、ZREM、ZSCORE
# 类型和多态
1. Redis的类型检查通过redisObject结构中的type属性实现。
1. Redis的多态性表现在：
    1. 有些命令可以对多种数据类型使用（如DEL、EXPIRE）
    1. 有些类型限定命令，可以对多种底层编码使用（如LLEN，对ziplist和linkedlist均可）

# 对象共享
1. 通过引用计数统计共享情况。一个对象可能被多个程序引用，也可能被多个键的值所引用。
1. 当存储的数据存在重复情况时，将会进行对象的共享
    - 如键A插入后，又创建了值完全一样的键B，则键B的值会直接指向键A的值，并增加引用计数
1. Redis默认创建存储0~9999的全部整数值的字符串对象。这些值将会直接用于共享。
    - 实际上，出于验证完全相等的性能考虑，Redis仅对包含整数值的字符串对象进行共享。

# 空转时长
1. 键的空转时长计算方式是：当前时间-值对象的lru时间

# redis-cli
1. redis的命令行控制工具。
1. 实用指令
    1. TYPE：获取一个键值对中值的数据类型
    1. OBJECT：
        - OBJECT ENCODING：查看一个键值对中值的底层实现类型
        - OBJECT REFCOUNT：查看一个键值对中值的引用计数
        - OBJECT IDLETIME：查看给定键的空转时长
1. 实用技巧
```bash
# 通过管道方式，批量执行控制命令
cat insert.txt | redis-cli --pipe
```