---
title: "《Hadoop权威指南（第四版）》读书笔记"
date: 2024-01-22T19:33:41+08:00
categories:
- 读书笔记
- 技术书
tags:
- Hadoop
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/hadoop-logo.jpg
draft: true
---
在大数据处理领域，Hadoop生态凭借多年的运营，基本上已经成为相关实现的事实标准。本文记录为读书笔记。
<!--more-->
## 准备工作
使用docker compose搭建：参考[Hadoop]({{<relref "/content/post/Tools/wsl.md#Hadoop">}})

## 写在前面
Hadoop是由Apache Lucene创始人Doug Cutting创建的。Doug Cutting一直致力于制作强大的搜索系统。他发起了Lucene（1999）、Nutch（2004）、Hadoop（2006）等项目。在发展过程中，Google的三篇论文（GFS、MapReduce、Big Table）起到了决定性的作用，Hadoop可以视为是三篇论文的最佳开源实现。

书中提到的[SETI@home](https://setiathome.berkeley.edu/)。是一个志愿计算项目，鼓励志愿者将自己的CPU的空闲时间利用起来，计算SETI项目分发的天文数据。这些数据往往需要进行几小时乃至几天的运算。和Hadoop的目标场景在多种维度上都有显著区别。

书中提到的一些示例程序，需要有相应的数据。可以从以下参考资料中获得。
- [提取并整理 NCDC 气象数据](https://zhuanlan.zhihu.com/p/556150264)。在使用ftp匿名登陆时，用户名为anonymous，密码输个邮箱就行。注意需要特殊网络连接（你懂的）。
    ```bash
    # 下载2023年全部数据
    wget -r -nH ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-lite/2023 --ftp-user=anonymous --ftp-password=你的邮箱
    ```
> Hadoop作为大数据处理生态解决方案。其生态中的各个组件也并不是简单接入就能适用于所有业务场景。在技术选型时，要充分对比各种方案。

## 一些坑：
1. Hadoop执行程序是需要打包程序，上传到集群上执行的，因此对于MapReduce程序，如果想要调试的话，是不能无配置就直接在本机调试（没有那些Jar）。但本地调试还是很重要的，参考[How to Set Up Hadoop on Windows: A Step-by-Step Guide](https://medium.com/@DataEngineeer/how-to-set-up-hadoop-on-windows-a-step-by-step-guide-37d1ab4bee57)、[MapReduce的本地运行模式（debug调试）](https://blog.csdn.net/qq_38200548/article/details/84057611)
    - 暂未成功
    - 另外也可以调研一下其他调试方式。
3. Hadoop的各类依赖在Maven中可能会产生依赖冲突，需要通过插件进行exclude解决。


## HDFS

## Yarn

## I/O操作

## MapReduce


## 生态
### HBase和Hive

### ZooKeeper

### Spark

### Flink

### Avro

### Flume

### Sqoop

### Pig

### Solr

### 其他

## 参考资料
《Hadoop权威指南 第四版（修订版&升级版》

