---
title: "设计模式-分布式系统篇"
date: 2023-02-19T22:51:52+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
分布式系统是大型系统最终的归宿，不一定会写，但也要了解。
<!--more-->
## 关键词
1. 基于数据库
1. 基于缓存
1. 基于zookeeper
1. 一致性模型
    - CAP定理
    - BASE 理论
    - 强一致性
    - 弱一致性
    - 最终一致性
1. 缓存数据最终一致性
    - 强一致性（两段提交和三段提交模型, Paxos或者Raft算法）
    - 最终一致性
1. 分布式事务解决方案
    1. eBay 事件队列方案
    1. TCC （Try-Confirm-Cancel）补偿模式
1. SOA
1. 微服务
1. Servless
1. 分布式系统架构设计
1. 分布式事务
1. 分布式锁
1. 分布式定时器
    - Quartz