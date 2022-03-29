---
title: "编程题笔记：总结篇"
date: 2022-03-29T10:28:30+08:00
categories:
- 求职
- 习题
tags:
- 持续施工
- 习题
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algo-exercises.jpg
math: true
---
本篇以记录实用算法和技巧为主，和题目记录对照使用。
<!--more-->
# 位运算技巧
- 位运算不保证跨平台通用，实际生产中使用时需要注意
- 无临时变量交换两个数的值：异或^
- 二进制下统计1的个数、去掉最低位的1：n&(n-1)
- 查表法：
- 更多技巧请参考：[位运算](https://blog.csdn.net/deaidai/article/details/78167367?utm_source=distribute.pc_relevant.none-task)、[位运算奇技淫巧](https://blog.csdn.net/holmofy/article/details/79360859)。