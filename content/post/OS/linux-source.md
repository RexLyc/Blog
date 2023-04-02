---
title: "边学边用linux-源代码篇"
date: 2022-12-10T12:54:37+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg

---
部分Linux源代码赏析
<!--more-->
## 常见宏
1. L1_CACHE_ALIGN(x)：返回x所指向的内存区域的起始cacheline的边界地址
1. ____cacheline_aligned：局部数据，将数据分配到程序.data段，起始位置对齐cacheline
1. __cacheline_aligned：全局数据，其他同上

## 参考
1. [Linux Kernel Map可视化](https://makelinux.github.io/kernel/map/)