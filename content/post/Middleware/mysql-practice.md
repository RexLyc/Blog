---
title: "MySQL：最佳实践"
date: 2022-04-02T23:35:09+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/mysql-logo.png
---
本文总结一些MySQL的最佳实践案例
<!--more-->
# 建库建表
1. 默认使用InnoDB、mb4utf8字符集
1. 不使用触发器等拖累MySQL计算资源的特性
    - 可以交给具体业务维护
# 查询
1. 不负向查询：not
1. 不模糊查询：like
1. 不允许where子句使用函数、表达式处理查询到的字段值
    - 可以对常量使用
1. 不select *
1. 多分片并带分片键进行查询
1. 不用大表join
1. 合理使用索引