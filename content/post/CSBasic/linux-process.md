---
title: "边学边用linux-进程管理"
date: 2022-04-01T14:12:14+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
进程管理是操作系统的重中之重，本文将对Linux的进程管理进行总结。
<!--more-->
## 进程状态
1. 状态类型：
    - R（TASK_RUNNING）：可执行状态
    - S（TASK_INTERRUPTIBLE）：可中断的睡眠状态
    - D（TASK_UNINTERRUPTIBLE）：不可中断的睡眠状态
    - T（TASK_STOPPED、TASK_TRACED)：暂停状态、跟踪状态
    - Z（TASK_DEAD、EXIT_ZOMBIE）：退出状态，僵尸进程
    - X（TASK_DEAD、EXIT_DEAD）：退出状态，即将销毁
1. 示意图
    <center>
    <img src="/images/Linux/process-state-change.jpg" style="max-width:70%">
    </center>
## 部分源代码
```cpp
// 部分字段
struct task_struct {
    volatile long state;
    unsigned long flags;
    int sigpending;
    mm_segment_t_addr_limit;
}
```