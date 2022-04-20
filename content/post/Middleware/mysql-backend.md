---
title: "MySQL：底层篇"
date: 2022-01-24T19:22:42+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/mysql-logo.png
---
想要良好的使用MySQL，对底层原理一定要有一定的掌握。本文致力于讲解实用的底层技术。
<!--more-->
## 客户端和服务器端
1. 连接
    1. 连接池：降低连接延迟、连接复用
## log
### Binlog
1. 参考：
    1. [研发应该懂的binlog知识（上）](https://www.cnblogs.com/rjzheng/p/9721765.html)
## 事务
1. 事务特性：ACID
    - A（Atomic）：原子性
    - C（Consistency）：一致性
    - I（Isolation）：隔离性
    - D（Durability）：持久性
1. 事务隔离级别：
    - 事务的不同隔离级别中，主要有三种情况不同，见下表
    | 隔离级别 | 脏读 | 不可重复读 | 幻读 |
    | --- | --- | --- | --- |
    | 读未提交（Read Uncommitted） | √ | √ | √ |
    | 读已提交（Read committed）| × | √ | √ |
    | 可重复读（Repeatable read） | × | × | √ |
    | 可串行化（Serializable） | × | × | × |
1. 锁机制
    
## 存储引擎
1. 索引：
    1. 优点：减少扫描量、避免表锁、随机IO变顺序IO
    1. 缺点：占用存储、单个存储元素修改时间变长
    1. 索引类型：主键索引、唯一索引、联合索引、普通索引
    1. 数据结构：B+树、Hash索引
1. 其他关键词：
    1. 多引擎支持
## 查询计划
## 多机
1. 分片（sharding）：
    1. 出现原因：
        1. 单一实例主db，扛不住写入了，拆分成多个主写入（多个机器）。
        1. 主从同步，虽然主可以并发写，但是同步一般是单线程，拆分之后可以提高同步效率。
    1. 实现方式：
        1. 在代理层做路由、将库拆分成多个部分，分散在多个机器上。（分片键修改困难、扩缩容困难、数据迁移困难）
1. 同步
1. 其他关键词：
    1. 多区域写入同步：
        - 建立双工链路
        - 用UTC时间戳解决一致性问题，选最新的
    1. 同城多机房（强一致性）：由代理层负责，写入只允许在主库，并同步到其他机房。读取一定在本机房。
## 高可用
1. High availability（HA）：高可用是MySQL进入生产应用的基本要求。
1. Orchestrator：一种高可用复制管理工具。
    - 基本功能
        - 管理集群拓扑
        - 监控运行状态
        - 机器切换
        - 状态通知
    - 参考：[Orchestrator介绍](https://www.cnblogs.com/zhoujinyi/p/10387581.html)、[官方Github链接](https://github.com/openark/orchestrator)