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
进程管理是操作系统的重中之重，本文将对Linux的进程管理，以及和进程调度、资源共享相关的各方面内容进行总结。
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

## 进程间通信
1. 按所处空间分类：
    - 内核空间：消息队列、信号
    - 用户空间：共享内存

## 锁
1. 锁的种类
    1. 自旋锁（SpinLock）：在目标资源被锁时不阻塞当前线程，而是本地轮询等待，适合用于临界区很小的业务，能避免线程切换带来的性能损失
    1. 互斥锁（Mutex）：在目标资源被锁时阻塞当前线程，解锁之后由系统负责唤醒
    1. 读写锁（RWLock）：共享互斥锁，读模式共享，写模式互斥，更为合理的互斥锁。但是需要考虑到读、写模式都可能面临饥饿，需要进行一定的策略来平衡读写需求
    1. RCU锁（Read Copy Update Lock）：更高级的读写锁，支持多读多写，实现复杂
    1. 可重入锁（可递归锁）：普通锁默认都是不可重入的，即同一线程在上锁后再获取同一个锁会陷入死锁，可重入锁则支持多次上锁
    1. 条件变量：需要和互斥锁一同使用，更偏向于用来等待其他线程确认某个条件达成，来唤醒目标线程
1. 锁实现的相关软硬件原理：
    1. 关闭中断：操作系统层面，在上锁时，需要考虑时钟中断等中断程序，因此为了避免多个线程同时竞争上锁（尤其是在上锁过程中出现中断），会进行**关闭中断**
    1. CAS和内存总线：硬件层面，指令上目前有CAS指令，可以**原子指令上锁**，此外还提供了多核处理器时，**锁内存总线**，以此来避免多核之间的竞争
    1. 缓存锁：锁内存总线显然造成很高的性能损失，明明只需要锁一个变量，现在所有CPU和内存都无法通信了，开发者基于CPU的内部缓存，进一步开发了**缓存锁**，即利用缓存行的MESI状态，结合**RFO指令**，完成多CPU之间的互斥性。但缓存锁存在失效情况，如锁定数据跨越多个缓存行（缓存的最小单位），或者处理器不支持缓存锁定。此时仍然只能锁定内存总线。
    1. 内存屏障：
        - 即使有了缓存锁的实现，还会有性能问题，因为RFO指令的执行并不是立刻，发出该指令的CPU只能等待，而接到RFO的指令又必须立刻放下自己的工作，对相应缓存无效化。为了解决这个性能问题，又引入了**Store Buffer和Invalidate Queue**，分别对应打算写数据和数据被无效化的情况，CPU发出RFO后不需要等待反馈，可以先将处理结果写入Store Buffer，只有RFO收到确认结果之后才会真正执行Store Buffer；同样的当接收到缓存无效指令，也会先放到Invalidate Queue中，等待CPU空闲时再执行。
        - 写屏障保证在写之前，所有的Store Buffer都执行，读屏障保证在读之前，所有的Invalidate Queue都执行。具体到锁上，上锁时会建立读屏障，保证临界区资源最新，释放锁时会建立写屏障，保证临界区资源更新写回。
        - 关于内存屏障，不同的语言提供了不同的保证，比如Java、C++各自提出了不通的解决方案，来实现一定程度上对内存一致性模型的使用。如使用松散的一致性要求（允许重排），或者使用严格的一致性要求（禁止重排）。对不同场景下LoadLoad（本指令之前的所有加载和之后的所有加载）、StoreStore（本指令前的所有写入和之后的所有写入）、LoadStore、StoreLoad需求进行控制。
    > 缓存锁和RFO指令也是导致伪共享的核心原因，当临界资源位于同一个缓存行内时，多核的竞争使用会造成反复锁定其他CPU，造成业务逻辑性能甚至差于单核串行处理
    > 由于现代编译器和CPU都会对执行顺序进行优化，因此内存一致性模型对并发控制中的锁机制的影响也非常大。不通的CPU硬件架构对内存一致性模型实现方式不同。

## 中断


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

## 参考
1. [Linux中的各种锁及其基本原理](https://www.cnblogs.com/TMesh/p/11730847.html)
1. [操作系统中锁的原理](https://www.jianshu.com/p/61490effab35)
1. [总线锁、缓存锁、MESI](https://blog.csdn.net/qq_35642036/article/details/82801708)
1. [多线程编程：伪共享以及其解决方案](https://blog.csdn.net/fenghuoliuxing990124/article/details/85127135)
1. [内存一致性模型WIKI](https://zh.wikipedia.org/zh-hans/%E5%86%85%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7%E6%A8%A1%E5%9E%8B)
1. [指令重排与内存屏障](https://www.cnblogs.com/ykpkris/articles/15785715.html)
1. [Java指令重排和Happens-Before原则](https://www.cnblogs.com/ITPower/p/13580691.html)
1. [C++内存模型和顺序一致性](https://www.cnblogs.com/fortunely/p/16739327.html)