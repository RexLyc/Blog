---
title: "分布式消息队列：合集篇"
date: 2023-03-16T22:01:17+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/dist-mq.jpg
---
分布式消息队列是大型分布式系统的重要中间件之一，甚至可能是最重要的，最终一致性经常是由消息队列保证，本文总结各种消息队列。
<!--more-->
## 概述
- 场景：买火车票（缓冲）、问答业务（多个部门要消费同一个消息）
- 作用：解耦上下游、缓冲、广播、持久化
- 选型：NSQ、Kafka
- NSQ组件：producer（通常通过http发送消息）、consumer（消费者，nsqd推送消息）、nsqd（服务器端主要组件）、topic（队列的逻辑概念）、channel（一个topic可以被多个channel消费）
    - 特性：requeue（处理失败的消息可以重新排进队列，超时情况也会）、defer（可以指定消息经过一段时间之后才被消费到）
- Kafka组件：producer、consumer（从服务器拉数据）、Broker（一个服务器节点）、Topic、Paritition、Consumer Group（一个topic的消息可以被多个group消费，每个group消费一个完整的消息）
- 部分实现细节：
    - NSQ结构：
        - nsqlookupd：nsqd会向指定的nsqlookupd注册自己的信息，consumer启动的时候去指定的nsqlookupd搜索自己想要链接的nsqd。之后直接连接。
        - disk queue：内存不够时写磁盘。
    - Kafka
        - Rebalance：zookeeper（herd effect羊群效应、脑裂问题）coordinator（更好的管理，join/sync group、metadata version、心跳）。
        - 高性能：batch send、压缩、ISR（Replica中一部分接收消息就认为成功）、append only、page cache、零拷贝技术
- 对比：
    -  各种常见MQ
        | MQ | 消息不丢 | 投递保证 | 顺序保证 | 可回溯 | requeue/defer | 推拉 | 吞吐量 | 生态 |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
        | NSQ | NSQD故障会丢 | 至少一次 | 无序 | 否 | 是 | 推送 | 一般 | 无 |
        | Kafka | 持久化 | 3种都有 | 支持有序 | 是 | 否 | 拉取 | 极高 | Hadoop |
        | RocketMQ | 持久化 | 至少一次 | 支持有序 | 是 | 可重试 | 包装拉取的推送模式 | 高 | Java |
        | RabbitMQ | 持久化 | 三种都有 | 无序 | 否 | 否 | 推送 | 较高 | Erlang |
    - 其中：nsq投递保证实际弱于at least once（因为不持久化）。无序的原因包括requeue、defer、写磁盘三种。nsq的主要数据都是存储于内存的。吞吐量其实也不低，但是和kafka比起来低很多。
- 思考：
    - 如何做到写DB和发消息的事务性
    - 如何让nsq尽量不丢消息？
    - 如何做到exactly-once（分布式系统唯二的难题：exactly once delivery和消息保序）

## Kafka
- Kafka是什么：分布式高吞吐量消息系统分布式流平台
- 设计理念：低延迟（以O(1)处理消息）、高吞吐、水平扩展（broker间的消息分区、分布式消费）、顺序性（Paritition内）、多场景（离线数据处理&实时数据处理）
- zookeeper：kafka中的zookeeper起到什么作用？建议看一下新版本，有没有人分享。目前应该是zk只和broker连接，负责存储少量的Kafka元信息（选举Broker的控制节点、broker探测）
- Topic & Paritition & Segment：
    - Topic：逻辑概念，发布-订阅使用
    - Paritition：一个Topic包含多个Paritition，每个Partition对应物理意义的一个文件夹，一般均匀分散到多个broker中
    - 写Partition：Append only，速度很快。
- Producer：默认异步发送（本地有一个发送队列），可以强行flush弄成同步。不保证强顺序性（队列中的发送内容可能先后有差别，可以设置max.in.flight.request.per.connection=1，则同一时间只发送一个消息来完全确保顺序性）。消息路由策略（可以根据自定义的key，选择发送的partition）。
- Consumer：2级API
    - low level（Assign）：指定目标Partition、起始offset、每次消费的消息长度、只消费特定消息
    - high level（Subscribe）：每个consumer属于特定的consumer group、offset由zk或kafka管理、实现rebalance、默认一个group顺序消费Topic的全部消息
    - Consumer Group Rebalance方案：
        - 自治式：Consumer启动时注册id到consumer group下，zk路径为/consumer/consumer_group_name/ids/consumer_id。在consumer-/group_name & /broker/ids & /broker/topics等目录下启用watch。【问题：羊群效应，broker/consumer变动会导致所有consumer进行rebalance。脑裂，consumer从zk上查看当前group情况并根据相同策略决定rebalance，但zk的特性决定了视图可能不一致。结果不可控，consumer不知道其他同组成员是否rebalance成功】
        - 集中化的rebalance：基于coordinator的rebalance。从zk监听topic、paritition变化。接收consumer的注册，为每一个group选leader，由leader来制定rebalance方案，leader通过发送SyncGroup请求将rebalance方案发给coordinator，其他consumer也用syncGroup从coordinator处获得方案。
- 高可用机制：【CAP：Partition Tolerance，分布式系统在遇到网络分区故障时，仍能保证对外输出满足一致性和可用性的服务（比如无脑裂等问题）】
    - 数据复制：producer只将数据发送给对应partition的leader（topic中的leader是partition级别的，每个partition有一个）。其他follower去leader处拉取数据。复制模式既不是同步也不是异步，采用了ISR机制，只要ISR中的都接收到，就算是接收消息成功，可以被消费。【常用一致性算法：WNR、Quorum、Paxos及其变种】
        - ISR会自动清除同步过慢的follower，极端情况下，所有的follower都被剔除，leader提交所有的消息。当follower跟上ISR后，也会自动加回来。
        - 一次Failover的情况：ISR中leader挂掉，随机选一个做leader。过一段时间恢复了，会先truncate（缩短）到已提交的数据，然后开始catch up，并回到ISR。
        - 更严重的问题：如果Replica全部挂了。需要一致性的话，还是需要等待ISR中任意节点恢复（可能整个Paritition不可用）。需要高可用的话，只需要选择第一个恢复的Replica，无论是否在ISR中（可能数据丢失）。
- Exactly Once：
    - 两阶段提交：耗时长、不适合低时延场景，需要相关系统都支持XA接口（XA是X/Open DTP组织定义的两阶段提交协议）。
    - at least once+下游幂等处理：实现相对简单
    - Offset更新和数据处理放在同一事务中：需要下游可回滚，以及相关事务系统参与。
- 思考：
    - 一个需要搜索的问题，为什么zookeeper被逐渐弃用？

## 参考
1. [分布式事务之深入理解什么是2PC、3PC及TCC协议？](https://www.cnblogs.com/wudimanong/p/10340948.html)
2. [ZooKeeper集群脑裂问题处理，值得收藏！](https://cloud.tencent.com/developer/article/1758883)
2. [分布式一致性Raft算法](https://zinglix.xyz/2020/06/25/raft/)
3. [关于etcd的一些谣言](https://ms2008.github.io/2019/12/04/etcd-rumor/)