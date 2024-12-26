---
title: "大数据和AI技术：框架篇"
date: 2022-11-16T14:24:29+08:00
categories:
- 计算机科学与技术
- 人工智能
tags:
- 人工智能
- 暂停更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/bigdata&ai.jpg
math: true
---
目前大数据工具集、机器学习工具集是做相关领域研发的必备工具。本文对实用内容进行简单总结。
<!--more-->

## 大数据中间件
### Hadoop生态概述
1. 概述：集群软硬件栈：便宜硬件 → OS（Linux） → DataNode & Node Manager → Job & Task。其中一些基础的模块，如
    - NodeManager（每台机器一个进程）
    - Yarn Resource Manger（一个集群一个）
    - Zookeeper Lock Service（Zookeeper提供的锁服务）
    - HDFS
        - Namenode（HDFS中重要的元信息维护节点，负责存储HDFS文件系统树、文件块映射、网络拓扑）
        - DataNode（文件块的最终存储位置，定期和NameNode通信）
1. HDFS
    - 特性：10K级别的集群
    - 一次写入：先向Namenode申请路径（元信息）、之后直接向对应的Datanode写入。
1. HBase：
    - 特性：1K级别的集群，多维度的排序表。自动分片、failover。适合超大规模的半结构化数据，弱一致性，仅支持单行事务。
    - 架构：HMaster存储元信息、管理切换Region Server。每个Region Server内存储一张排好序的表（或者一部分）。底层基于HDFS。
    - 一次读写：也是向HMaster查询表的位置、进一步直接向Region Server查找数据。
1. Yarn：
    - 特性：集群计算&存储资源管理器、10k级别集群、调度监控分布式任务、提供统一的程序管理接口（Spark、Flink等）
    - Yarn框架下的一次Job：client向Resouce Manager提交，Resource Manager选择一个合适的机器，启动其Container为App Master负责这一次的Job，并启动其他若干container，由该App Master负责，进行Job的具体执行。
1. MapReduce：
    - 特性：固定2阶段Map&Reduce、大规模数据处理、容错性好、扩展性高
    - 劣势：所有中间结果都要落盘、
1. Spark：
    - 特性：大数据计算引擎、in-memory（并提供了挂掉的计算节点数据恢复的方法）、可以用函数式编程直接编写、10~100倍于mapReduce的速度。
    - 数据模型：RDD（Resilient Distributed Dataset），一种分布于多个节点的数据结构。一个节点上的数据成为一个partition。RDD之间的计算关系用DAG图保存，关键节点会落盘。
        - API：无需shuffle（map、filter、union），需要shuffle（groupByKey、reduceByKey、repartition）
        - 创建：从hdfs路径、从kafka接口、从其他RDD（通过计算）
        - 痛点（所有分布式计算都有）：shuffle问题
            - shuffle问题由数据依赖性产生（Narrow&Wide），糟糕情况下可能需要落盘
            - shuffle情况出现时，将RDD流程切分为多个stage。
1. Flink：
    - 特性：流式处理（也有批处理但是一般不用）引擎、大规模、高并行、低延时
    - 往往使用滑动窗口确定一次处理的数据范围。（eg:每次5s，滑动1s）
### 分布式实时计算系统
1. 特点：和离线（批量）计算对比：强调实时性、数据集无界、增量计算、常驻计算
1. 实时计算引擎：Spark-streaming、Storm、Flink
1. 一次Flink Job的组成：
    - Flink Program
    - Job Manager（对应Yarn的Application Master）：负责将一个Job的各task创建task manager（并分配硬件资源）
    - Task Manager：负责执行具体task
    - Task Slot：真正的执行单元，一个task slots是一个thread，做了内存隔离，但是多个task slot可以在一个task manager所在机器运行。
1. task由一群operator chains组成。流入算子+计算+流出算子。

### Hive

### Yarn
- 作业抽象：RM在接到一个job时，检查发送心跳过来的NodeManager，选一个记录下来（分配给这次job），在该Nodemanager上启动一个AppManager，**未完成**
- RM剖析：交互（对外）、安全、状态机（RMApp、RMAppAtempt、RMContainter、RMNode）、AM管理（ApplicationMasterLauncher、AMLivelinessMonitor、ApplicationMasterService）、NM管理（）、应用管理（）
- NM剖析：Container管理
- 资源隔离：一台NM上运行多个Containter时，需要进行资源隔离。
    - cpu通过cgroup隔离，最小保证机制
    - MEM通过NM的ContainterMonitor服务进行监控，对于超过使用限制的Container直接杀死


## 深度学习
### Tensorflow
### Keras
### 预训练模型
1. 网站：
    - TensorFlow Hub
    - Keras：[Keras Applications](https://keras.io/api/applications/#available-models)
    - NVIDIA NGC

## 参考
1. [HDFS NameNode内存全景](https://tech.meituan.com/2016/08/26/namenode.html)
1. [Spark 什么是RDD？](https://www.jianshu.com/p/6411fff954cf)