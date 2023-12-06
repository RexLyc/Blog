---
title: "算法导论其十：进阶篇"
date: 2023-12-06T11:04:53+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algorithm.jpg
math: true
draft: true
---
本章节将会聚焦一些经典问题，以及经典进阶算法。在追求高效率的路上是没有极限的。
<!--more-->
## 经典问题
### LCA最近公共祖先
- ToDo

### RMQ区间最大值最小值查询
- Tarjan的Sparse-Table算法
    - 构建：$dp[i][j]$表示i为起点，长度为$2^j$的区间的最值。状态转移方程（可以理解为二分）为$dp[i][j]=min/max(dp[i][j-1]+dp[i+(1<<(j-1))][j-1])$。注意二层循环，j为外层（按照区间长度分治）。
    - 查询：寻找小于等于区间范围的区间（或多个区间，可以重叠）。
- 线段树：$O(n)$建立，$O(logn)$查询
- 笛卡尔树+LCA+DP：$O(n)$建立，$O(1)$查询