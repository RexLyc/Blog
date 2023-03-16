---
title: "Go：合集篇"
date: 2023-03-15T22:33:17+08:00categories:
- 计算机科学与技术
- 编程语言
tags:
- Go系列
- 合集篇
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/go.jpg
---
Go是近些年流行的后端开发语言之一，尤其擅长高并发程序，本文总结基础语法和开发知识
<!--more-->
## 基础语法
- 有特色的三个常用类型：slice、channel、interface{}
    - 数组：类型确定、长度不可变。（强类型甚至到数组长度也是类型的一部分）。值拷贝。
    - slice：point + len + cap。
        - var xxx []int
        - 从数组构造：slice:=data[1:4:5] // low:high:max。high和max都是尾后下标。
        - copy(dst,src)。
    - channel：读c:=<-ch，写ch <- v，关闭close，创建make。
        - 长度为0的不带buffer。不发生额外拷贝，读在写之前。长度大于0的带buffer，写在读之前。
        - 一种用法：for c:= range xxxChan {}
        - 只读channel：<-chan xxxType
        - 只写channel：chan<- xxxType
    - interface
        - 抽象类型，唯一确定的是具有某些方法
        - 经典接口：
            - type Reader interface { Read(b []byte) (int,error) }
            - type Writer interface { Write(b []byte) (int,error) }
        - 意义：封装、解耦依赖。
- 并发：
    - 多考虑使用Context控制并发：
        - 接口类型，4种方法Deadline、Done（<-chan struct{}）、Err（表示context的返回情况或原因）、Value(内部有个map)。
        - 通过继承、形成Context树，保证线程安全。
        - 本质上就是方便进行线程间的数据传递（从上向下传递），以及线程结束控制。带cancel的，cancel执行时会使所有子孙context的Done信号触发。Deadline/Timeout则是根据设置自动触发。
        - context.TODO / Backgroud。
        - https://zhuanlan.zhihu.com/p/110085652 可以看看具体实现。主要是emptyCtx、valueCtx、cancelCtx的各自实现。
    - WaitGroup
    - 锁比channel高效，但要考虑好使用的场合（大量数据同步的时候才有必要）
    - 线程池（避免协程爆炸）
- Profile：go tool pprof
    - CPU分析、Mem分析
- 高效GO：
    - 减小对象创建、减少堆内存分配
    - sync.Pool：
    - sync / atomic
    - 对象复用（比如Read接口的设计，用的是参数，而不是返回一个[]byte）
    - streaming：以序列化网络传输为例。如果用全部内容一次性做序列化，再压缩发送，先后创建若干个临时变量（可能还很大），如果这种传输频繁，频繁创建大对象，gc压力大。可以改成流式的方式，按照一个确定的大小，反复复用一个对象。
- 看视频过程中学到的:
    - os.OpenFile(“name”,os.O_RDONLY|…,os.APPENDMODE|…)。
- 分清struct的定义和赋值和赋值式定义：var xxx XStrucct，xxx := XStruct{}。定义的时候不能用{}。
- go协程和进程线程的对比
    - 协程不会做到完全公平的调度，依赖于协程自身让出控制权（举个例子就是如果开启较短的函数打印东西，可能main推出了，协程还没运行，什么也不会打印）

## GC
- GO GPM调度模型：G（代码）、M（执行）、P（执行所需权限和资源）
    - 一个G就是一个Go Routine，runtime中用类型g表示。一个routine退出时，g将会放到空闲g对象池中以便后续routine使用
    - 一个M是一个系统的线程，系统线程可以执行用户的go代码、runtime代码、系统调用、或者空闲等待。同一个时间可能由任意数数量的M（当一个M阻塞的调用系统调用时，会将M和P解绑，并创建新的M执行P上的其他G）
    - 一个P代表了执行go代码需要的资源，比如调度器状态、内存分配器状态。在runtime中用类型p表示。P的数量精确的等于GOMAXPROCS。（P可以理解为调度器的CPU、p类型可以理解为每个CPU的状态）
    - 所有的g、p、m都在堆上，永不释放。
    - 每个G有个用户栈，初始2k，可伸缩。每个M有个系统栈（如果是unix系统，还有一个signal栈），大小固定。
- GC：
    - 算法：STW  标记清除法  三色标记法  混合写屏障机制
    - 牺牲了gc的吞吐量来换取极端的stw时间。
    - 阶段对比：
        - GCMark：标记准备阶段，为并发标记做准备工作，启用写屏障，STW
        - GCMark：扫描标记阶段，与赋值器并发执行，并发
        - GCMarkTermination：标记种植阶段，保证一个周期内标记任务完成，关闭写屏障，STW
        - GCoff：内存清扫阶段，将需要回收的内存归还到堆中，并发
        - GCoff：内存归还阶段，将过多的内存归还给操作系统，并发
    - 标记阶段CPU默认至多25%的P，gcAssist不受此限制（申请内存前帮gc干一点活，防止申请快于标记）。P设置错了会有问题。混合写屏障，着色成本高。
    - GC影响：服务抖动（p99上涨）、请求超时
- 优化技巧：
    - 不建议获取协程id，因为会导致协程无法gc
    - 什么时候做优化：优化的价值大于投入的时间价值、写代码时、有benchmark进行对照。
    - 如何找优化点：通过pprof，看newobject的情况
    - 避免频繁分配大量小对象、避免指针数量多
    - slice预分配内存优化（slice在append时会2倍增长，拷贝很耗时）
    - map预分配优化（减少内存拷贝和rehash）
    - 使用strings.Builder而不是bytes.Buffer（同时使用预分配能更快）
    - string和slice byte强转不要乱用（string设计上是immutable的，强转后实际使用相同的内存，对其写入的结果，是未定义的）
    - 函数中尽可能使用值而不是指针（使用指针会使逃逸分析将变量分配在堆上，无法像栈上变量一样被自动释放掉，需要进入gc）。只有大对象才有使用指针的必要。
    - 接上条，slice类型内，如非必要，不要包含指针（大量数据逃逸到栈上）
    - map存值而不是指针
    - 使用sync.Pool优化内存。（但要非常清楚对象的生命周期，sync.Pool表现就像C中的malloc和free）
    - 使用struct{}。（比如需要用一个map去重，map[int]struct{}要优于map[int]bool，编译器做了优化，所有空stuct指向同一个地址，不占用空间）
    - 使用atomic优化（锁一般是系统实现、atomic可达到硬件实现）
    - 使用不带缓冲区的channel（因为可以减少内存拷贝），channel底层还是有很多锁的，尽量用来通知而不是传值。
    - 关注逃逸分析（所有channel传递的内容都会逃逸到堆上进行空间分配）、查看哪些变量逃逸到堆上，并考虑是否ok
- 其他：
    - 自己写marshal接口。
    - 使用goroutine池（gopkg/gopool）
    - 使用并发安全的rand库（gopkg/rand）
    - 根据P做shared
    - 加入padding，防止false-sharing？


## 参考
1. [Golang之协程详解](https://www.cnblogs.com/liang1101/p/7285955.html)
1. [Golang协程调度](https://blog.csdn.net/sunxianghuang/article/details/105033936)
1. [Golang在runtime中的一些骚东西](https://www.purewhite.io/2019/11/28/runtime-hacking-translate/)