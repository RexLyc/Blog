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
## 概述
1. 架构关键词：
    - 客户端层面：连接池、shell
    - 服务器层面：服务器进程、NoSQL接口、SQL接口、解析器Parser、优化器Optimizer、缓存和缓冲区、存储引擎、文件系统
1. 适用场景：需要事务支持、高并发需求、数据一致性要求较高、时延和稳定性有一定要求

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
    > 注：不可重复读侧重于对单行数据的修改，幻读则是说明了此时有行增加删除，不仅需要行锁，还需要区间锁甚至表级锁才能避免
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
## MVCC
- 多版本并发控制机制（Mutil-Version Concurrency Control MVCC），用于实现事务隔离级别的底层机制，以更好的支持对数据库的并发访问
- InooDB的实现：
    - 基本原理：每行添加两个值，修改该行的事务id（trx_id）、指向上一个版本的指针（roll_pointer），再结合事务执行时创建的ReadView（主要包含当前事务id、活跃事务id列表），通过不同的ReadView生成策略，完成对读已提交和可重复读的区分支持
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

## 参考
1. [Mysql中MVCC的使用及原理详解](https://www.cnblogs.com/shujiying/p/11347632.html)
1. [对MySQL中MVCC理解](https://baijiahao.baidu.com/s?id=1629409989970483292&wfr=spider&for=pc)
1. [MVCC原理之ReadView](https://juejin.cn/post/7154701807694905351)