---
title: "面试总结：大全篇"
date: 2023-02-11T22:15:25+08:00
categories:
- 求职面试
tags:
- 求职面试
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/interview.jpg
---
这篇笔记总结历次求职面试的经历，对笔试、面试题目进行总结，也会对非专业考察内容进行一定的梳理。
<!--more-->
## 专业性考察
### 数据库相关
1. 聚簇索引优势在哪儿？辅助索引为什么使用主键作为值域？
    - 由于行数据和叶子节点存储在一起，这样主键和行数据是一起被载入内存的，找到叶子节点就可以立刻将行数据返回了，如果按照主键Id来组织数据，获得数据更快。
    - 辅助索引使用主键作为"指针" 而不是使用地址值作为指针的好处是，减少了当出现行移动或者数据页分裂时辅助索引的维护工作，使用聚簇索引就可以保证不管这个主键B+树的节点如何变化，辅助索引树都不受影响。

## 非专业内容考察

## 参考
1. [知乎：后端都要学习什么？](https://www.zhihu.com/question/24952874/answer/518162706)
1. [MyBatis官网](https://mybatis.org/mybatis-3/zh/index.html)
1. [MySQL8.0 存储引擎综述](https://dev.mysql.com/doc/refman/8.0/en/pluggable-storage-overview.html)