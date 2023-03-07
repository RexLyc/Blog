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

## 参考
1. [360 安全规则集合](https://github.com/Qihoo360/safe-rules)