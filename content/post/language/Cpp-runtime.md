---
title: "C/C++：运行时要点"
date: 2024-04-28T14:40:40+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
draft: true
---
和本系列其他文章不同，本章节主要讲述的是C/C++在运行时的一些要点。堆栈、寄存器、异常、指令等。
<!--more-->
本文的计划偏向于底层，阐述C/C++在运行时是如何组织内存，使用CPU指令来完成任务的。强调运行时的动态视图。

## 运行时分析工具
GDB：GDB的能力不用再多说了，可以用gdb调试运行时的各种信息，宏展开，数据结构，寄存器，堆栈信息。

libunwind：libunwind的目标是做一个通用，而且高性能的C风格API，来提供异常处理、堆栈调试、内省、高效setjmp等功能。参考[libunwind官方网址](https://www.nongnu.org/libunwind/docs.html)

