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
## 重要模块
### 时间
1. 基础关键词：
    - timekeeper：内核中用于组织和时间相关的数据结构
    - clocksource：一个在指定输入频率始终下工作的计数器，系统中可以存在多个clocksource，对应不同的硬件时钟
    - clock_event_device：时钟事件设备，clocksource只能计数，需要本设备进行编程，指定在特定的计数情况下触发事件
1. jiffies：以定时器中断为记录点，用于记录系统启动之后经历的tick次数。一个tick就是一次时钟中断。内核中使用CONFIG_HZ来配置每秒的中断次数。注意
    - 该时间显然存在回绕情况，在一定时间周期之后，将会从头开始
    - 由于是以定时器事件的时间间隔为基准进行记录，是低精度时钟
    - 基于jiffies的低精度定时器：时间轮（time wheel）机制
        - 基本原理：
            1. 在定时器中断事件产生时，检查到期的定时器
            1. 为了提高效率，每个CPU会独立地管理属于自己的一批定时器
            1. 添加定时器时，会根据到期时间和所分配的CPU的jiffies计数的差值，计算出idx，并根据idx将该定时器添加到tv1~tv5中合适的列表内。tv1是最先到期的，tv5是最晚到期的。
            1. tv1中放置的是在接下来256个jiffies内即将到期定时器列表，此时只需对当前CPU真实jiffies值取后8位作为索引，就可以在每次时钟中断时，去除对应位置的定时器列表，调用其定时器
            1. 对tv2~tv5中的定时器，每当CPU的jiffies计数在对应的区间有进位时，意味着有些定时器需要从后面的tv移动到前面
            > 时间轮的名字非常贴切，定时器移动的方式就像是时钟的齿轮一样，从慢到快依次传递。该算法在大部分时间里提供了O(1)的查询到期定时器的能力，但在发生定时器tv组间迁移时，仍然可能退化到O(N)
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
                // 指向当前CPU需要处理的定时器列表
                struct timer_list *running_timer;
                // 当前CPU经历过的jiffies值
                unsigned long timer_jiffies;
                // 指向下一个即将到期的定时器
                unsigned long next_timer;
                // tv1-tv5用于对定时器进行分组
                struct tvec_root tv1;
                struct tvec tv2;
                struct tvec tv3;
                struct tvec tv4;
                struct tvec tv5;
            } ____cacheline_aligned;

            struct tvec_root {
                struct list_head vec[TVR_SIZE];
            }

            struct tvec {
                struct list_head vec[TVN_SIZE];
            }
            ```
1. hrtimer：高精度定时器，提供纳秒级定时精度，用红黑树实现，树的最左侧节点就是最近到期的定时器。
    - 系统启动时可能并不处于高精度模式，此时时钟中断事件仍由jiffie机制接管。在切换到高精度模式后，会接管时钟中断事件，但是仍然会想办法为传统jiffie机制提供时钟事件模拟。
    - 高精度时钟在实现过程中会使用更为精确的clocksource和对应的clock_event_device。
    - hrtimer的到期回调有三个使用方式：在没有使用高精度时，仍然在jiffie的时钟中断事件中处理；在HRTIMER_SOFTIRQ软中断中处理；使用高精度模式后，在每个clock_event_device到期时间中处理
## 参考
- [init systemd](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
- [init service systemctl区别](https://blog.csdn.net/lineuman/article/details/52578399)
- [Linux时间管理系统](https://blog.csdn.net/droidphone/category_1263459.html)