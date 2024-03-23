---
title: "学点业务：推荐系统"
date: 2024-03-18T21:37:58+08:00
categories:
- 业务
- 推荐系统
tags:
- 交叉
- 推荐系统
# thumbnailImagePosition: left
# thumbnailImage: //example.com/image.jpg
draft: true

---
本文记录推荐系统的基本组成和常见设计优化，偏向后端研发。
<!--more-->
## 基本构成
召回：
1. 准备所有推荐的内容（内容池，这个池子已经是根据用户相关的了）
2. 过滤

> 这一步是并行，可能做多种召回，一起输入到粗排

粗排：
1. 个性化打分
2. 截断

精排
1. 个性化打分
2. 截断（100个左右）

> 粗排和精排，都同样可能是并行，请求多个模型，或者请求多个

重排
1. 多样性的选择策略
2. 生成最终候选集

完成以上链路工作的就是推荐引擎，对引擎来说，最重要的输入就是三个：场景、用户、视频Item。其中
- 以用户属性来请求视频，可以看作是一个倒排索引
- 对于算法工程师，提供算子：上下文、用户信息、视频信息

里面涉及到的偏后台开发实现的一些细节
1. 索引，倒排索引
2. 曝光过滤，避免重复推荐。（布隆过滤器）
3. 模型部署方案。粗排、精排中的机器学习模型
4. 推荐模型中的大量特征存储。
   1. parameter server：[参数服务器](https://cloud.tencent.com/developer/article/1694537)
5. 如何满足实时性要求，比如10w qps

## 术语
1. 协同过滤：基于用户特征的（推送有相同爱好的用户的物品）、基于物品特征的（推送和当前物品相关的物品）
2. 冷启动：新用户、新物品
3. 

## 核心
### Parameter Server
参考：[深入浅出之「Parameter Server」架构](https://cloud.tencent.com/developer/article/1694537)、[Bilibili 分布式机器学习系统: Parameter Server 设计](https://www.bilibili.com/video/BV1vY4y1r71u)

## 优化点
### 索引
基础方案，实时+全量+分布式协调。

### mmap


## 其他
### 分层A/B实验

## 应用场景
1. 什么时候需要推荐系统？
   1. 用户缺乏明确目的
   2. 长尾信息缺少流量
   > 相反，很强的工具属性的软件，就不一定需要做推荐系统
2. 

## 参考
1. [知乎-卢新来：​倒排索引](https://zhuanlan.zhihu.com/p/338487179)
2. [知乎-卢新来：推荐系统架构的“三足鼎立”](https://zhuanlan.zhihu.com/p/92743599)
3. [B站强推！王树森老师亲授推荐系统！](https://www.bilibili.com/video/BV1jh411A7WZ)
4. [百亿数据，毫秒级返回，如何设计？--浅谈实时索引构建之道](https://www.cnblogs.com/xiekun/p/14613073.html)
5. [美团广告实时索引的设计与实现](https://tech.meituan.com/2018/05/11/adp-rtidx-ls.html)
6. [智能后端和架构-推荐系统](https://www.yijiyong.com/ac/rsintro/01-intro.html)