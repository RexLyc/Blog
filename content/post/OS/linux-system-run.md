---
title: "边学边用linux-系统运行篇"
date: 2022-02-22T21:47:16+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
了解系统运行是更好的使用Linux的基础。本文讲述Linux的系统运行原理。
<!--more-->
## 启动
1. 流程
    1. BIOS
    1. GRUB
    1. 内核启动
    1. 服务启动：
        - systemctl是systemd风格的服务启动方式。用于替换init风格的服务启动方式。
## 子系统
### 时间子系统
1. jiffies：以定时器中断为记录点，用于记录系统启动之后经历的时间。内核中使用CONFIG_HZ来配置每秒的中断次数。但，
    - 该时间显然存在回绕情况，在一定时间周期之后，将会从头开始
    - 由于是以定时器事件的时间间隔为基准进行记录，是低精度时钟
1. 低精度定时器：
    - 基于jiffies：时间轮（time wheel）机制
        - 基本原理：
            1. 在定时器中断事件产生时，检查到期的定时器
            1. 为了提高效率，每个CPU会独立地管理属于自己的定时器
            1. 
        - 相关代码
            ```c
            // #include<linux/timer.h>
            struct timer_list {
                // 定时器事件链表
                struct list_head entry;
                // 定时器到期时刻（jiffies值）
                unsigned long expires;
                // 所属cpu的tvec_base结构
                struct tvec_base *base;
                // 回调函数
                void (*function)(unsigned long);
                // 回调参数
                unsigned long data;
                // 允许的到期后执行延迟
                int slack;
                // ...
            }

            struct tvec_base {
                spinlock_t lock;
                struct timer_list *running_timer;
                unsigned long timer_jiffies;
                unsigned long next_timer;
                struct tvec_root tv1;
                struct tvec tv2;
                struct tvec tv3;
                struct tvec tv4;
                struct tvec tv5;
            } ____cacheline_aligned;
            ```
## 参考
- [init systemd](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
- [init service systemctl区别](https://blog.csdn.net/lineuman/article/details/52578399)
- [Linux时间管理系统](https://blog.csdn.net/droidphone/category_1263459.html)