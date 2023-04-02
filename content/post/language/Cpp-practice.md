---
title: "C/C++：最佳实践"
date: 2023-03-03T11:00:31+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 滚动更新
draft: true
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
本篇记录一些C/Cpp开发中的一些最佳实践。
<!--more-->

## STL
1. 智能指针：
    - weak_ptr：A、B类内各用weak_ptr保存对方，在其他地方使用A、B时可以随意使用shared_ptr、unique_ptr

## 实用工具
1. 一款能够查看C/C++编译的汇编代码和原始代码对比情况的工具：[godbolt.org](https://godbolt.org/)，也有[github仓库](https://github.com/compiler-explorer/compiler-explorer)可以自行部署

## 参考
1. [360 安全规则集合](https://github.com/Qihoo360/safe-rules)