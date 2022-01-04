---
title: "编程题笔记"
date: 2021-12-30T16:29:39+08:00
categories:
- 求职
- 习题
tags:
- 持续施工
- 习题
# thumbnailImagePosition: left
# thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
题目内容来自leetcode、牛客等平台，偶有面试笔试真题。
<!--more-->
# 总结
1. 惯用语言为C++，偶尔会使用Java、Python。
1. 由于牛客等平台在面试时，不一定会提供头文件，需要自己牢记使用的标准库函数位于哪个头文件中。
1. 简单：中等：困难 = 2：4：1。
1. 算法分类：
    1. 搜索（BFS、DFS）：单目标搜索、多目标搜索，搭配数据结构优化，单调性优化
        - 搜索从思路上可以分为：生成法、判别法两种。前者生成的一定是一个解，后者生成所有，并进行判定。
    1. 动态规划：记忆化的搜索。
        1. 单状态dp[][][]，多状态dp1[]、dp2[]
        1. 多种决策可能
        1. 递推形式
        1. 区间dp，最优化区间
        1. 背包问题
        1. 空间优化，[时间优化参考](https://www.cnblogs.com/flipped/p/9669202.html)
    1. 贪心
    1. 分治：可分性、合并时的复杂度
    1. 排序：排序后是否可贪心
    1. 数据结构
        1. 单调栈、单调队列
        1. 双堆
        1. 数组：双指针、预处理、和递归结合
        1. 树：前中后序的递归&非递归写法，层次遍历
    1. 图算法：并查集、拓扑排序、欧拉图、最短路、网络流
    1. 模拟
    1. 数学推导
    1. 字符串：匹配算法、正则匹配、通配符匹配、转为图算法
    1. 滑动窗口
1. 一些建议：
    1. 从时间空间约束上寻找提示
    1. 从暴力法入手，观察搜索树结构，逐渐优化
# 力扣

# 牛客

# 面试题

# 笔试题

# 经典书籍
1. 《程序员面试经典》

1. 《剑指Offer》

# 语言相关要点
1. C++