---
title: "设计模式-并发编程"
date: 2022-03-28T21:43:30+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 暂停施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
并发编程是现代计算机程序的基本构成部分之一。本文将总结常见的相关内容和常见问题。
<!--more-->
## 并行和并发
1. 并发（Concurrency）：具有一段时间内同时处理多个任务的能力。
    - 并发和顺序（Sequential）相对应。顺序只能做完一件，再做另一件。不能都处于运行阶段。
1. 并行（Parallelism）：具有在一个时间点上同时处理多个任务的能力。
    - 并行和串行（Serial）相对应。串行只能在一个时刻处理一个任务。不能同一时刻同时执行多个任务。
1. 并行实际上是并发的子集。本文以并发为主要总结目标。

## 并发模型
1. 从内存共享角度：
    1. 基于内存共享的：数据直接在内存种共享，临界区需要使用同步控制，有锁或无锁
    1. 基于消息传递的：进程间通讯或者内存映射
1. 基于消息传递的模型：
    1. Actors模型：强调通信的工作实体，不同的Actor之间通过消息发送和接收进行直接通信，清楚的知道消息的来源和目的地，除此之外，不能再对对方暴露任何和自身属性、行为相关的接口。Actor之间也没有接触点，其消息发送是异步且解耦的，接收Actor可以根据自身状态决定是否接收等。理论上需要一个无限大的区域进行缓冲。代表语言/库有：Erlang/Akka。
    1. CSP（Communicating Sequential Processes）顺序通信模型：强调通过信道传输消息，发送者和接收者并不知道对方是谁。但CSP的消息交换存在同步的接触点（即读取和写入的位置）。代表语言有：Go。
    > Dont't communicate by sharing memory, share memory by communicating. (R. Pike)

## 常用并发编程模式
1. 
## 常见问题
1. 伪共享/假共享（False Sharing）：先看一段受害者代码
    ```cpp
    void f(unsigned __int64 begin, unsigned __int64 end, unsigned __int64 &result) noexcept {
        result = 0;
        for(unsigned __int64 i = begin; i < end; ++i) {
            result += i;
        }
    }

    void paraAdd(unsigned __int64 begin, unsigned __int64 end, unsigned __int64 &result) {
        unsigned int core = thread::hardware_concurrency();
        unsigned __int64 step = (end - begin) / core;
        vector<thread> threads;
        unsigned __int64 *results = new unsigned __int64[core];
        for(unsigned int i = 0; i < core; ++i) {
            threads.push_back(thread(f, begin + i * step, begin + (i + 1) * step, ref(result[i])));
        }
        for_each(threads.begin(), threads.end(), mem_fn(&std::thread::join));
        result = 0;
        for(unsigned int i = 0; i < core; ++i) {
            result += results[i];
        }
        delete[] results;
    }
    ```
    - 这段代码看似将求和分给多个线程进行计算。但实际上，求和总用时甚至高于单一线程求全部和。
    - 多核处理器之间虽然共享内存，但是由于CPU缓存（行缓存）的存在，实际上某一核修改数据时，需要花费一定的时间，才能使其他核的缓存同步。当这种同步频繁出现时。共享实际上已经失去了意义。
    - 可以通过给数据填充无效字段，来避免行缓存中出现其他核会修改的数据，进而提高并发性能。但这并非最好的方案。还是应当从设计上尽量避免这类粗暴的可同时读写的大段内存共享。
    - volatile并不能完美解决问题，因为这相当于放弃了缓存，性能还是会慢。
# 参考
- [计算机内存模型](https://blog.csdn.net/weixin_48024348/article/details/113926049)
- [伪共享](https://zhuanlan.zhihu.com/p/124974025)
- [两种常用的并发模型:CSP和Actor](https://blog.csdn.net/sixdaycoder/article/details/90751972)