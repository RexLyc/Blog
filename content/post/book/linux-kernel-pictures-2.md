---
title: "图解Linux内核 读书笔记（下）"
date: 2025-10-22T20:43:15+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统
- 施工中
- 读书笔记
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/linux-kernel-pictures.jpg
draft: true
mermaid: true
math: true
---
本文是下半部分：进程管理和信号。
<!--more-->
# 进程
进程是一个非常复杂的话题，涵盖调度、通信等多方面。

## 数据结构
先来看进程的数据结构。内核定义了`task_struct`，这个结构体实在是太大了，这里给一下[链接](https://elixir.bootlin.com/linux/v6.2.16/source/include/linux/sched.h#L737)。


<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/200/  在线阅读 -->

<!-- https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt -->

<!-- https://elixir.bootlin.com/linux/v6.2.16/source -->

<!-- 本文为第二部分，即电子书201页之后，纸质书189页之后。 -->