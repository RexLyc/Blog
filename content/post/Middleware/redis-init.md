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
## 概述
1. 截至2022年2月，redis是一个BSD许可下（允许商用，只需要附上原许可证）的内存数据库。
1. redis的设计目的就是性能至上。常被用做数据库，缓存，消息存储（message broker）。
1. 提供超多种类的数据结构。
1. 支持事务、不同的缓存策略、持久化、集群化等功能。
1. 采用[ANSI_C](https://zh.wikipedia.org/wiki/ANSI_C)编写。
1. 由于基于《Redis设计与实现》一书（Redis2.9）编写本博客，因此需要特别注意浏览新特性一章。
## Redis速览
- redis是一个nosql数据库（这是主要定义）。虽然平常用来做缓存。
- 常用的缓存中间件：Redis（以及官方发布的分布式Redis-Cluster）、Codis（以Redis为基础的分布式缓存中间件）、Memcached（老牌缓存中间件）
- 缓存作用：性能提升、数据存储（短生命周期）、功能性（地理位置数据等）
- Redis-cluster架构：Redis Master & Slave，集群状态用Gossip Protocol通信。Master和Slave是人工指定的。【非中心节点式结构】
    - key是如何分布的：每个节点分布一定数量的槽。物理节点上则会映射到[0,16383]，将Key做hash映射到这些操。槽可以在机器间迁移。
    - 可用性：自动分配slave、自动迁移slave。
    - 关于槽：https://www.cnblogs.com/rjzheng/p/11430592.html 。
- 跨区域同步（跨洲同步）方案之一：多个单独的Redis集群，用2个queue做双主同步。
- redis 6.0新功能：
    - 多线程IO【5.0开始，接入层就逐步开始多线程、但事件层仍是单线程】
    - client-side-caching（客户端缓存）
    - REPS3协议
    - 官方redis cluster proxy【当前的中间件设计基本原则，用proxy保护实际集群接口】
- redis数据类型：
    - string（普通kv）、List（列表）、Set（集合）、Hash（哈希）、ZSet（有序集合，有hash功能又可以根据字段进行排序？？）、Hyperloglog（distinct value，非精确性计算）、geo（地理位置数据、即ZSet的包装）、stream（持久化队列）
    - 相关操作：String(set/get)、List（push/pop）、Set（sadd/sremove，增量式expand）、Hash（hset/hget，增量式expand）、ZSet（zadd/zremove）、超时设置（expire/expireat）仅针对顶层元素执行超时处理、incre（增/减数值，注意不影响超时）
- 操作：
    - Redis multi：统一执行多个命令，具有事务性。redis-cluster环境下，如果key跨越多个分片，直接返回失败。
    - 超时处理：【重要内容、等待学习】
- 建议：
    - Redis持久化成本还是很高的，尤其是海量数据时。更建议使用ABase。
- 题外话：
    - Codis团队后来独立出来，做了TiDB（和ByteKV结构很像）。
    - 摩拜的redis服务PAAS化经验：k8s+Codis+负载均衡+无人值守方案
## 基本原理
- 定义：Redis（Remote Dictionary Server），开源的内存、分布式、键值数据库，具有可选的持久化能力。
- 场景：计数器&统计数据（前100，显示名词）、任务队列（定时任务、推送）、关注（关注和粉丝列表）
- 本课程按照单线程去讲，但Redis高版本需要再学习一下
- 关键技术：
    - 事件循环：Redis启动后会进入一个巨大的while循环。大致分为文件事件和时间事件。具体循环：before-sleep阶段 → process events阶段(file events & time events) → stop / start / end【before sleep  →  epollwait  →  事件处理  →  看看定时任务是否到期并处理 】
    - 文件事件（file event）：对多个客户端实现多路复用，将客户端请求结果返回。【read事件，新建连接&接受请求。write事件，返回结果。】
    - 时间事件（time event）：负责处理定时任务来维护数据库状态。记录那些要在指定时间点运行的事件，多个时间事件以无序链表的形式保存。【更新服务器统计信息、数据库后台操作、关闭&清理客户端连接、检查是否RDB dump&AOF重写、主从同步、集群信息同步】
- 细节：一次AOF写（和重写不一样），是属于before sleep阶段（比定时事件更频繁）。
- 部分细节：
    - before sleep：进入event loop前执行。包括集群状态检查（ok & fail），处理被阻塞的client（一些阻塞式请求，如blpop，即block pop，阻塞式弹出队尾），将AOF buffer持久化到AOF文件。【减少大key，减少耗时命令】
    - redis逐出：当执行write达到内存上限时，强制将一些key删除。
- 策略（整体分两种，此外具体算法还有3种）：allkeys、volatile（设置了过期的key）。LRU（最久未被使用）、random（随机）、ttl（最快过期的）
- 特点：不是精准算法、抽样对比。每次写入操作前判断，逐出是阻塞请求的。
    - redis过期：当某个key达到ttl时间，认为该key已经过期
- 策略：惰性删除（读写前判断，过期则删除）、定期删除（redis定时时间中随机抽取部分key判断ttl）。两种策略同时存在（前者为了数据最新、后者为了保持合理的内存占用）。
- 特点：并不会精确的按照设定时间过期、定期删除策略中会判断过期比例（只有低于一定阈值才会结束，否则一直抽样并删除）
- 建议：打散key的过期时间，避免key过期时间集中
    - 持久化：将内存中的数据dump到磁盘文件。RDB、AOF两种。
- RDB持久化：经过压缩的二进制，fork一个子进程来做（一瞬间卡顿）
- AOF持久化：保存所有修改数据库的命令，先写aof缓存，再由before sleep阶段进行写aof文件。
    - AOF重写：AOF文件达到一定大小后触发，也会fork一个子进程，创建一个新AOF文件，将修改数据库的命令压缩（比如5次+1 → 1次+5），写完之后替换AOF文件。
    - 主从复制：
- 模式：主从节点都可以挂从节点，保证最终一致性
- 同步：全量同步（传递RDB文件&restore命令重建kv、传递rdb dump过程中追加的output积压缓存）、部分同步（根据offset传递积压缓存队列中的部分数据），之后进行命令扩散
    - ![redis同步](/images/middleware/redis-sync.png)
- 总的来说就是：主节点对每个从需要单独保留一个output缓冲区。而主节点上的积压队列只有一个，是所有从节点共享的。
    - client-output-buffer-limit：代表output缓冲区大小（256MB硬限制，超过该大小断开连接，意味着同步的速度跟不上主节点更新的速度，另有64MB 60s限制，也会断开连接）
    - repl-backlog-size（复制积压大小）：每当master发送写入给slave，会在积压队列中写入一份，如果从节点断开连接，但是重连后发送的master runid匹配，并且offset仍在积压队列中，将对应的offset后的数据，复制到对应slave的output缓冲区，发送并完成部分同步。
    - 无限同步问题：如果全量同步时，tps过高，超过output-buffer-limit限制而断开，重连后问题可能仍然存在。另外tps过高时，如果slave在加载rdb期间，无法执行master的同步写入命令，复制积压过多（环形队列），导致offset失效，再次进入同步。
- BGSAVE能确保数据在fork时间点前后分隔开的底层原理，父子进程之间的COW机制（Copy on write），由操作系统保证bgsave一瞬间的rdb是固定的。
- 可以配置从节点在同步过程中仍然保持工作（使用旧数据），但是重载数据并catch up的过程一定会暂停服务。
    - 请求：
- pipeline：client将多个命令存起来，缓冲区满了在发送，redis（proxy+server）处理完一个就返回一个（proxy到server也是缓存多个请求一起发送）。
- mget：一个命令中有多个key，等所有key全部完成后返回。proxy端会等待&缓存所有redis的回复再返回给client。
- 建议：利用pipeline替代mget，且控制一次请求的量（50个以内）。
    - 集群：
- 一致性hash：proxy作为中心节点，管理不同的redis节点（都是master）。明显的问题就是水平扩容时，会发生hash命中问题，造成数据丢失。
- redis cluster（基于gossip协议）：最大的问题就是两两通信造成网络负载上升，在节点达到1k左右的时候，会出现一些一致性上的问题。
    - 其他部署问题：
- 客户端的连接池&负载均衡
- 客户端failover，断路器
- cache&db架构的延迟删除：原因（cache与db的数据不一致，一次删除后再发生get请求，会触发读db从，但此时db主从同步可能尚未完成）、解决方法（大于1次主从延迟再删一遍）
- 双机房延迟删除：为了保证双机房cache的一致。需要把内容发送给其他机房的队列中再删一次。本质上还是要依赖db的删除动作完成cache的一致性。

## 系列分章
1. 数据结构和用法
1. 基本原理
1. 集群用法
1. 其他
1. 新特性
## 参考文献
1. 《Redis设计与实现》2014年版
1. [redis官方文档](https://redis.io/documentation)
1. [超时删除](https://www.cnblogs.com/lukexwang/p/4694094.html)
1. [Redis过期策略](https://www.cnblogs.com/java-zhao/p/5205771.html)
1. [Redis主从同步机制详解](https://blog.csdn.net/Lyong19900923/article/details/102700158)
1. [redis主从同步（replication）详解](https://blog.csdn.net/fuyuwei2015/article/details/70922729?depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)