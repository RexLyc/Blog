---
title: "边学边用linux-系统运行篇"
date: 2022-02-22T21:47:16+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 滚动更新
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

## 内核态和用户态
1. **To Be Continue**

## 中断
- 中断请求（Interrupt Request），缩写为IRQ，是发送到处理器的信号，可以临时停止正在处理的任务，转而去运行中断处理程序。中断分为硬中断和软中断两种方式，其对比如下
| 性质 | 硬中断 | 软中断 |
| --- | --- | --- |
| 嵌套 | 高优先级可以抢断低优先级 | 只能被硬件中断抢断 |
| 子类型 | 可屏蔽中断、不可屏蔽中断 | |
| 发起者 | 硬件设备（经由中断控制器） | 软件（进程）|
| 流程位置 | 上半部，追求尽快结束 | 下半部，处理硬中断未完成的耗时任务 |
- 基础关键词：
    - IRQ编号：系统中每一个注册的中断源，都有一个的编号用于标记
    - 触发：硬件信号可能存在多种，电平触发、边沿触发
    - 中断流控：正确控制连续发生的中断，尤其是同种中断将要发生嵌套时，一般是将中断控制权交给驱动程序处理
    - /proc/irq、/proc/softirq、/proc/interrupt：运行时可以查看中断处理相关情况
    - NMI：不可屏蔽中断，比较特殊，该中断发生时CPU必须无条件响应，不能挂起，不能屏蔽
- 基本流程
    1. 硬件中断：
        1. 系统启动阶段初始化相关硬件，尤其是中断控制器
        1. 配置各软件部分，驱动程序申请中断编号，
        1. 设备发出中断信号
        1. 中断控制器判断中断是否响应，以及通知哪些CPU
        1. 某个或某些CPU响应该中断，进入中断入口
        1. CPU通过中断控制器的中断请求获取到IRQ，将该处理流转到中断流控，取出合适的中断响应函数（或线程）
    1. 软中断：
        - 在软中断触发之后，将会依次
            1. 关闭CPU中断，避免竞争
            1. 修改软中断位图（bitmap），标记当前有对应的软中断需要处理
            1. 如果当前不在中断上下文中，可以唤醒守护进程，否则只能等待当前中断上下文执行到退出阶段
            1. 恢复CPU中断
        - 软中断处理
            - 当调用local_bh_enable函数，激活本地CPU软中断，满足条件下会调用do_softirq来处理软中断
            - 当硬中断处理完成后调用irq_exit时，会调用do_softirq来处理软中断
            - 当守护线程ksoftirq被唤醒时，处理软中断
            > 前两种情况，在do_softirq中会循环处理，但是如果超出一定次数限制，则会将任务转交给守护线程执行，保证性能
- 部分代码
    ```c
    //判断当前CPU的中断状态
    #define in_interrupt() (irq_count())
    #define irq_count() (preempt_count() & (HARDIRQ_MASK | SOFTIRQ_MASK | NMI_MASK))

    // -------------------------- 硬中断 --------------------------
    // 驱动程序申请中断注册
    int request_threaded_irq(
        unsigned int        irq,                //申请的IRQ编号
        irq_handler_t       handler,            //中断上下文处理器
        irq_handler_t       thread_fn,          //中断回调线程
        unsigned long       irqflags,           //中断标志
        const char          *devname,           //中断名称
        void                *dev_id);

    // 可能使用连续的irq描述结构体数组，简单粗暴
    #ifndef CONFIG_SPARSE_IRQ
        struct irq_desc irq_desc[NR_IRQS] __cacheline_aligned_in_smp = {
            [0 ... NR_IRQS-1] = {
                .handle_irq = handle_bad_irq,
                .depth      = 1,
                .lock       = __RAW_SPIN_LOCK_UNLOCKED(irq_desc->lock),
            }
        };
    #else
    // 也可能使用基数树（radix tree），可以动态分配，可以编号不连续
    struct irq_data {
        unsigned int    irq;            //系统irq号
        unsigned long   hwirq;          //硬件irq号
        struct irq_chip *chip;          //
        void            *handler_data;  //irq私有数据
        void            *chip_data;     //中断处理器私有数据
        cpumask_var_t   affinity:       //CPU处理映射关系
    };
    struct irq_desc {
        struct irq_data     irq_data;
        irq_flow_handler_t  handle_irq;         //中断流控制
        struct irqaction    *action;            //中断响应链表（多个设备可能共享同一个irq）
        raw_spinlock_t      lock;               //保护irq_desc自身的锁
        const char          *name;
        wait_queue_head_t   wait_for_threads;   //等待当前irq所有线程
        // ... 其他
    };
    #endif

    // -------------------------- 软中断 --------------------------
    struct softirq_action {
        // 用于回调的函数指针
        void (*action)(struct softirq_action*);
        void *data;
    };
    // 软中断的类型数量很有限，而且也不建议增加
    enum {
        HI_SOFTIRQ=0,
        TIMER_SOFTIRQ,
        // ... 一些
        TASKLET_SOFTIRQ,        // 微任务，建议用户广泛使用这个处理自定义任务
        SCHED_SOFTIRQ,          //
        HRTIMER_SOFTIRQ,        // 高精度时钟软中断
        RCU_SOFTIRQ,            // RCU操作软中断
    };
    static struct softirq_action softirq_vec[NR_SOFTIRQS] __cacheline_aligned_in_smp;

    //触发软中断
    void raise_softirq(unsigned int nr);
    ```

## 重要模块&机制
### CFS
### CGroups
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
- [软中断和硬中断](https://www.jianshu.com/p/52a3ee40ea30)
- [Linux中断子系统](https://blog.csdn.net/droidphone/category_1118447.html)
- [深入理解Linux中断机制](https://heapdump.cn/article/4514433)
- [Linux软中断过程详细总结](https://blog.csdn.net/Luckiers/article/details/123868625)
- [linux内核学习资料大全整理-Github](https://github.com/0voice/linux_kernel_wiki)