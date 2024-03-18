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
    - T（TASK_STOPPED、TASK_TRACED）：暂停状态、跟踪状态
    - Z（TASK_DEAD、EXIT_ZOMBIE）：退出状态，僵尸进程
    - X（TASK_DEAD、EXIT_DEAD）：退出状态，即将销毁
1. 示意图
    <center>
    <img src="/images/Linux/process-state-change.jpg" style="max-width:70%">
    </center>

## 进程调度
1. 进程调度是现代分时操作系统的核心功能之一。Linux从2.6.23开始引入CFS（Completely Fair Schedule）公平调度机制。
1. 基础关键词：
    - 调度器：负责实际完成进程调度过程的模块
    - 调度策略：不通进程可能采取不通的调度策略，调度器针对进程所使用的策略，判断是否将其放入可运行队列。实际实现中往往会结合多种策略。
        - CFS策略：目的是尽量让所有进程能占有的CPU时间保持一致
        - FCFS（First Come First Service）策略：先来先服务，不基于时间片调度，只要未主动退出或阻塞，一直占有CPU
        - RR（Round-Robin）策略：时间片轮转，进程按入队顺序分别执行自己的时间片，时间片到期、阻塞则移动到队尾
        - SPF策略：短进程优先，按照估计时间，优先执行短进程，只要未主动退出或阻塞，一直占有CPU
        - 多级反馈队列策略：分为多个优先级不同的队列，最高优先级的时间片最短，新任务添加到最高优先级队列中，以FCFS执行。如果无法在指定时间片内完成，则移动到下一优先级队列队尾，只有当高优先级队列空了，才以FCFS运行下一优先级队列
        - 优先级调度算法：具体分为非抢占式优先级、抢占式优先级两种。非抢占式允许进程一直运行，结束后再调度其他高优先级进程。抢占式则是只要有更高优先级进程出现，直接调度更高优先级进程开始运行。
    - 操作系统区别：
        - Unix：单纯的分时操作系统，使用时间片轮转的多级反馈队列策略
        - Linux：分为实时进程和普通进程两种，实时进程采用带有静态优先级的FCFS、RR，尽量保证实时性，普通进程使用动态的优先级调度。实时进程也优先于普通进程。
        - Windows：使用时间片轮转，基于优先级的抢占式调度
    - Linux进程级别：SHED_FIFO、SHED_RR、SHED_NORMAL这三种进程使用各自的调度策略
    - 组调度：由于部分进程可能会占用大量CPU，而对其他进程不公平，可以将相关的一组进程设置为组调度，此时这一组的进程将会被看作是一个调度实体（即一个进程）。尤其是在多用户操作系统中常见，避免A用户的使用影响到B用户的体验。
1. CFS的实现：
    1. 时间记账：
        - 相关数据结构：task_struct（进程信息）、sched_entity（进程调度相关信息）、ofs_rq（红黑树）
        - 基本原理：
            1. 计算虚拟运行时间vruntime，根据进程的nice值，计算出当前进程的时间片权重，并累加到vruntime上
    1. 进程选择
        - 红黑树：按照vruntime为键值构造的，最左节点就是vruntime最小的节点，也是最应当被优先调度的
            - 添加和删除进程的过程就是将进程节点添加到红黑树中的过程
            - 所有在红黑树中的节点，都是拿到时间片就可以立刻运行的。阻塞的进程会被移除。
    1. 调度器入口
        - 调度时机：并不是所有时机都适合调度，比如时钟中断是常见的调度机会，但调度并不能在中断上下文中进行，很多事件都可以对调度标记位进行标记，但调度执行的机会一般仅包括：
            - 系统调用返回到用户空间
            - 中断返回到用户空间
            - 中断返回到内核空间，需要内核支持内核抢占，不过内核抢占基本上已经是目前 linux 的默认配置
            - 重新使能内核抢占时
            - 进程主动放弃CPU
        - schedule()函数，其内部的主要流程有
            1. 禁止抢占，避免调度嵌套。关闭local apic中断，避免和中断竞争
            1. 针对上一个进程的处理，如果被阻塞，则需要进行移除
            1. 选择下一个执行的进程，会根据各种调度策略优先级进行查找，比如此时空闲进程数量不等于cfs调度器管理的进程数量，则意味着有实时进程存在，则优先遍历实时进程所用的各类调度策略
            1. 对上一个进程的进程信息进行最后的处理，更新其运行时间等数据，写回到红黑树中，如果是组调度，则会对同组的所有进程进行处理和修改
            1. 将进程现场保存，各寄存器写回内存，将新进程相关数据加载到寄存器。做一次内存屏障，将虚拟内存相关数据切换成新进程。恢复抢占，开启local apic中断，将控制权交给新进程
    1. 睡眠和唤醒
        - 当进程进入阻塞条件后，调度器会将其从红黑树中移除
        - 当进程等待的阻塞条件满足后，调度器会将其重新加入红黑树。对于可中断的休眠情况，其等待的条件更宽泛。

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

## 信号
linux开发中，在进程中对信号的处理是非常重要的。信号是一种软中断，信号的设计继承自Unix，因此有不可靠信号（SIGRTMIN前，信号可能丢失），可靠信号（后面定义的信号，支持信号队列，排队发送），在系统中可以通过```man 7 signal```，查看系统中和信号相关的内容。

信号相关的操作
1. 发送：```raise(int sig)```，```kill(pid_t pid, int sig)```
2. 接收：```sighandler_t signal(int signum, sighandler_t handler)```，更推荐```int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact)```
3. 处理：用户定义的任何满足```typedef void(*sighandler_t)(int)```函数签名的函数
4. 忽略：忽略将会调用默认处理。
5. 特例：SIGKILL（杀死）、SIGSTOP（暂停）不能忽略，不能捕获。

常见的信号
1. SIGHUP ：终端结束信号
1. SIGINT ：键盘中断信号（Ctrl - C）
1. SIGQUIT：键盘退出信号（Ctrl - \）
1. SIGPIPE：浮点异常信号
1. SIGKILL：用来结束进程的信号
1. SIGALRM：定时器信号
1. SIGTERM：kill 命令发出的信号
1. SIGCHLD：标识子进程结束的信号
1. SIGSTOP：停止执行信号（Ctrl - Z）
2. SIGSEGV：无效的内存访问

信号处理函数应当满足```可重入要求```
1. 不能使用外部的任何全局变量，以及其他可能会使用到这些变量的函数
2. 不调用malloc和free
3. 不调用标准I/O函数（可以，但不推荐）

也因此，我们可以实现对于一些崩溃情况的恢复，避免单一线程挂掉，进程直接挂掉
```c
// 自定义信号处理函数示例
#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
#include <setjmp.h>
static jmp_buf buf;
// 自定义信号处理函数，处理自定义逻辑后再调用 exit 退出
void sigHandler(int sig) {
    // 不建议，这里仅作测试
    printf("Signal %d catched!\n", sig);
    // exit(sig);
    longjmp(buf,1);
}
int main(void) {
    signal(SIGSEGV, sigHandler);
    int *p = (int *)0xC0000fff;
    if(!setjmp(buf)){
        *p = 10; // 针对不属于进程的内核空间写入数据，崩溃
    }
    else
    {
        printf("exit safely\n");
    }
    return 0;
}
```

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
1. [linux进程调度](https://dreamgoing.github.io/linux%E8%BF%9B%E7%A8%8B%E8%B0%83%E5%BA%A6.html)
1. [操作系统中常用的进程调度算法](https://blog.csdn.net/fuzhongmin05/article/details/55802925)
1. [linux调度子系统8 - schedule函数](https://zhuanlan.zhihu.com/p/363791563)
1. [Linux组调度原理](https://zhuanlan.zhihu.com/p/400102565)
2. [登龙（DLonng）Linux 高级编程 - 信号 Signal](https://dlonng.com/posts/signal)