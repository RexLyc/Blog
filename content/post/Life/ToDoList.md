---
title: "写在前面"
date: 2022-03-01T20:00:00+08:00
categories:
- 博客搭建
- 序言
tags:
- 计划
- 网站建设
- 滚动更新
# 置顶方式是添加权重
weight: 1000
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/todo.jpg
---
这里有我的序言，一些记录的问题，还有一些更新计划。
<!--more-->
## 序言

2021年，也就是在我刚刚开始坚持写博客的时候，那个时候还没有chatGPT，也没有通义千问，当时我的想法很简单，就是去记录一下自己平时解决问题的过程中，学到的知识，以及一些解决方案。也是当时苦于国内搜索引擎的搜索结果太差。不过这一切问题都随着大模型的出现，有了更好的解决方案。

所以现在我觉得浪费太多时间在撰写博客上，并不是一个很明智的选择了。很多问题，通过和大模型的互动都可以得到解决。对于不能解决的问题，可能才有记录的必要。而且即使这些问题，有可能过几个月，随着模型的升级，也会被解决掉。

那么这里到底记录些什么，目前（2024.12）我也还没想好。可能对我来说，这里将会变得更像一个日记本。记录一下自己的学习、思考、开发过程。底层的实现可以交给AI，所以对于个人，更重要的应该就是思考能力、创意这些了。

未来博客的更新思路就是：少即是多。提供原理和设计上的一些思考记录，目的是看完能快速了解。对于技术的细节内容，则在工作生活中，更多的交给AI来生成了。

对于已有内容，也会进行整理删减

## 计划更新方向
1. 操作系统、虚拟化、计算机网络
2. 软件设计学习
3. 实用工具和技巧（AI回答不好的部分）
4. 其他兴趣内容

## 更新列表
1. [ ] 删除/归档博客中不再有阅读意义的文章内容，甚至是文章
1. [x] 一个测试框架
    - [x] 开发语言和图形接口选择（建议前后端分离）
        - vue+js+electron：[DrawFlow](https://github.com/jerosoler/Drawflow)
        - 后端python/cpp/java
    - [ ] 可视化测试进度
    - [ ] UI测试，同步、异步接口的测试处理
    - [x] 测试中的异常处理
    - [ ] 网络化、批量化
    - [ ] 添加重做撤销功能（可持久化数据结构）
2. [ ] 技能准备
    - 中间件学习：ElasticSearch、HDFS & HBase、Hadoop & MapReduce & HIVE、Flink、Impala
    - 中间件复习：Redis、NginX、MySQL、Kafka
    - 分布式：一致性哈希、Raft、分布式事务
    - 容器：Docker、k8s
    - 工具：perf、gperftool、gprof、gdb、flamegraph
3. [ ] 博客填充
    - 所有内容较少的页面，尽量填充（也就是说快点把该学的东西补上来）
    - 在复习的过程中，补齐应有的图片
4. [ ] 基于UE5的地铁数字孪生系统
    - 智能维修、[爆炸动画](https://docs.unrealengine.com/5.1/zh-CN/setting-up-xr-explode-animations-in-unreal-engine/)
5. [ ] 把PC版的页面文章宽度增加一些
6. [ ] 算法分析，所有的分析方式（尤其是势函数法）
7. [ ] 跑通一个跨端WebRTC Demo。PC + Web。
8. [ ] 把菜谱章节更新一下。
9. [ ] 找一下二胡的乐谱，偶尔练一练。

## 更新一
1. [x] linux-shell - 8h
2. [x] spring1 - 8h
3. [x] design-pattern-init - 16h(各种模式找一个样例代码)
4. [x] git - 4h
5. [x] cpp-compile - 24h
    - [x] 编译链接装配
    - [x] linux gcc & windows vs
    - [x] gdb
    - [x] cmake
6. [X] cpp-standard - 16h
    - [X] C & C++
    - [X] C++ 98 03 11 14 17 20
7. [x] qt-all-in-one - 8h(后续可以再补充，尤其是针对嵌入式、针对模块的单独使用)
8. [x] jmeter-all-in-one - 2h(稍加补充)
9. [x] Unreal Engine5 入门 - 48h
10. [x] ibmmq-all-in-one - 2h(稍加补充)
11. [ ] linux-file - 30h
12. [ ] linux-system-run - 16h 
    - https://www.liuvv.com/p/c9c96ac3.html
    - https://blog.csdn.net/lina_acm/article/details/79767414
13. [ ] linux-process - 30h
14. [x] spring2 - 8h(稍加补充)
15. [ ] cicd-jenkins - 2h(稍加补充，后续可以更新不同项目的一些build的坑)
16. [ ] mysql-backend - 16h(整理之前的笔记)
17. [ ] mysql-command - 4h
    - https://www.csdn.net/tags/OtDaIg1sNzA2Ni1ibG9n.html
18. [ ] mysql-practice - 8h(整理一些sql实用脚本)
19. [ ] fullstack-jsts-basic - 8h
20. [ ] fullstack-jsts-aporia - 8h
21. [ ] design-concurrency - 24h
22. [ ] design-pattern-terms - 2h(后续不断补充)
23. [ ] redis-single - 8h
24. ~~[ ] WebLab-init - 30h(快点做完吧球球了)~~
    ~~- 可以看看BliKVM，以KVM形式做一个控制器~~
25. [ ] algo1 - 4h(重看一遍视频，把这部分博客补了)
- 合计：270h，预计需花费90天
## 更新二
1. [ ] 权限系统相关：
    - [ ] RBAC等模型
    - [ ] OAuth2.0
2. [ ] 计算机网络进阶
    - [ ] dhcp原理
    - [ ] dns
    - [ ] 代理和反向代理
    - [ ] 网络安全（firewalld、selinux），常见攻击手段和防御方式
3. [ ] java-构建（maven、gradle）
    maven 常见的几个解决办法：
    1. [ ] 仍然使用自带maven（有的时候用自己下载的高版本maven就是有问题）
    2. [ ] 使用解决冲突的一个插件，一般来说是exclude一些不要的包
4. [ ] java annotation：一些注解处理中常用的设计模式
5. [ ] linux-mod：内核模块开发
    - lsusb、modprobe、dkms
6. [ ] 大数据
    - [ ] pearson相关性系数、xgboost、随机森林、sklearn、lgboost、linear regression、交叉验证
    - [ ] 数据标注和预处理
7. [ ] 运维
    - [ ] google sre、docker、k8s
8. [ ] 面试
    - [ ] orm框架原理（mybatis）
    - [ ] rabbitmq & kafka
9. [ ] C++配套设施
    - [ ] 内存泄漏检测（-fsanitize=address）

## 长期型
1. 参加一些线下技术论坛、创业论坛
1. read&write (可以写一些有意义的活动心得、感悟)
1. 玩一下历年的年度游戏
2. 学英语，主要是听力、词汇量

## 待读书清单
- 计划必读
    - [x] Unreal Engine 4 Scripting with C++ Cookbook（快速阅读）
    - [ ] 大象无形：虚幻引擎程序设计浅析
    - [ ] Modern Cpp Tutorial: C++ 11/14/17/20 on the fly
    - [ ] Docker 容器与容器云（第2版）
    - [ ] Kubernetes中文指南
    - [ ] Prometheus操作指南
    - [ ] kubernetes 源码剖析
    - [ ] 深入剖析kubernetes (kubernetes deep dive)
    - [ ] 深入理解go语言
    - [ ] go专家编程（第二版）
    - [ ] 深入理解linux网络
    - [ ] containerd原理剖析与实战
    - [ ] 云原生数据中心网络
    - [ ] 网络虚拟化技术详解 NFV与SDN
    - [ ] kubernetes网络权威指南 基础、原理与实践

- 其他：
   - [ ] 3D数学基础：图形和游戏开发
   - [ ] Physically Based Rendering: From Theory to Implemention 3rd
   - [ ] 点石成金：访客至上的Web和移动可用性设计秘笈 原书第3版
   - [ ] SRE谷歌运维解密
   - [ ] 面向模式的软件架构
   - [ ] 黑客与画家
   - [ ] 分布式系统原理与泛型
> 参考：[推荐工程师合适读本](https://github.com/0voice/expert_readed_books)

## 网站建设
1. [ ] 设置固定最小宽度，在此宽度之下，必须水平滚动，不再进行换行
2. [ ] 左侧侧边栏css不对（图标不够展开时,870px左右，应当隐藏）