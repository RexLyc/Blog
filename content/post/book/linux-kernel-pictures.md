---
title: "图解Linux内核 读书笔记"
date: 2025-01-01T16:28:33+08:00
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
---
本书提供了大量的插图，来学习Linux内核。
<!--more-->

> 阅读期间，使用[linux kernel](https://kernel.org)，下载版本6.12.7

> 本书有很多细节，汇编代码，因此也建议作为科普，工具书阅读。有需要的时候可以回来看看。

## 概述和基础知识
1. 内核代码结构：
    - Documentation：文档
    - arch：和体系结构有关的，或者是其他模块中需要区分体系结构的内容
    - kernel：核心部分，包括进程调度、中断处理、时钟等，和体系结构相关的会放到/arch/xxx/kernel
    - drivers：驱动
    - mm：内存管理，同样也会有在/arch/xxx/xx下
    - fs：文件系统，一种文件系统拥有一种子目录
    - ipc：进程间通信
    - block：块设备管理
    - lib：内核空间下的通用函数库
    - init：内核初始化
    - firmware：由外部设备的芯片运行的固件程序
    - scripts：内核配置脚本
    - 其他（本书不涉及的）：net、crypto、certs、security、tools、virt（虚拟化）
1. 基础数据结构
    - Linux目前仍以C为主，所以其数据结构，以struct、和container_of等宏的方式形成，不像其他面向对象的语言提供的那种数据结构的形式
    - 一对多的描述方式：将链表结构嵌入到有需要的数据结构中
        ```c
        // 方式一
        struct branch {
            struct list_head head;
            // other member
        };
        struct leaf {
            struct list_head node;
            // other member
        };
        struct list_head {
            struct list_head *next, *prev;
        };

        // 方式二，节省空间但有一些不便
        struct hlist_head {
            struct hlist_node *first;
        };
        struct hlist_node {
            // pprev是前一个hlist_node的next指针的地址
            // 即若prev => curr, curr->pprev == &prev->next
            struct hlist_node *next, **pprev;
            // other member
        };
        ```
    - 多对多的描述方式：将多对多联系，抽象为connection结构体，再嵌入到有需要的数据结构中
        
        示例，这只是一个选课场景（学生-老师）下的示例，表示c语言具备的多对多抽象能力。
        实际场景中，设备-设备处理程序，就是一个类似的多对多的关系。
        ```c
        // 书上的示例描述了一个老师-学生的多对多场景
        // 一个学生可以有多个老师，一个老师可以有多个学生

        // 具有同一个老师的某些学生
        struct s_connection {
            // 连接到下一个s_connection
            struct list_head node;
            // 指向一个学生
            struct student *student;
        };
        
        // 具有同一个学生的某些老师
        struct t_connection {
            // 连接到下一个t_connection
            struct list_head node;
            // 指向一个老师
            struct teacher *teacher;
        };

        // 因为师生关系一定是双向的，所以可以将两种联系合并, 代表一个学生-老师的联系
        struct connection {
            // 其他具有相同老师的connection
            struct list_head s_node;
            // 其他具有相同学生的connection
            struct list_head t_node;
            // 当前老师
            struct teacher *teacher;
            // 当前学生
            struct student *student;
        };

        // 最后，每个学生和老师的数据结构
        struct teacher {
            // 连接到connection中的s_node
            // 遍历该s_node的connection，可获得所有的student
            struct list_head head_of_student_list;
            // other member
        };
        struct student {
            // 连接到connecton中的t_node
            // 遍历该t_node的connection，可获得所有的teacher
            struct list_head head_of_teacher_list;
            // other member
        }
        ```
        <!-- TODO：可考虑插入P13的图片 -->
1. 设计模式：注意内核的设计方式，是面向对象的
    - 模板方法模式：Template Method。即开发者实现固定的借口，系统会根据流程进行调用。
    - 观察者模式：内核中，xxx_listener、xxx_notify
1. 中断
    
    广义的中断可进一步细分为中断（interrupt）和异常（exception）。更进一步的，中断分为可屏蔽和不可屏蔽，都是来自I/O设备的。异常则是程序主动进行的，包括陷阱、故障和终止。不论是哪种，CPU只会在一个指令执行完成后再检查，**不会在执行中检查**。

    中断处理需要软硬件分工合作。CPU提供了处理的指令、以及相应的寄存器位来存储。

    中断处理程序是整个处理过程，从保护现场、处理、恢复现场。其中需要开发人员做的，一般是中断服务例程。涉及到两个关键的结构体：`irq_desc`、`irqaction`，是一对多，因为一个中断是可以被共享的。一个irq号对应一个`irq_desc`，会有通用的handler，而一个`irqaction`则代表一种设备更具体的处理，会有自己的handler供`irq_desc`中的handler调用。

    Top Half和Bottom Half，一些函数可看到th、bh的后缀，代表前半段、后半段。中断处理应当快速，头半段不能做复杂的处理。复杂逻辑应当使用工作队列、软中断，或启动单独的线程工作。

    注册`irq_desc`和`irqaction`的方式。
    ```c
    int request_threaded_irq(unsigned int irq, irq_handler_t handler,
        irq_handler_t thread_fn, unsigned long flags, const char *name, void *dev);
    
    int request_irq(unsigned int irq, irq_handler_t handler,
        unsigned long flags, const char *name, void *dev);
    ```

    这里有一个具体的例子，键盘和鼠标等外设，可以是共享相同irq（例如200）的设备，但是二者并不会直接拉起一次中断处理程序，会是由GPIO再发起一个irq（例如50），GPIO的设备将会负责相应的`irq_desc`的处理。键盘和鼠标只需要完成自己的对应200的，`irq_desc`，`irq_action`。共享中断需要该设备的驱动能够区分出来，是否是自己设备发出的，如果不能，那么不能进行共享。

    中断处理还有很多细节：比如中断处理时，又有新中断发生（一般来说是会继续处理最新的中断，如果有多个新中断，会丢失中间的）；是否还有软中断需要处理；中断处理结束后，需要返回内核态还是用户态等