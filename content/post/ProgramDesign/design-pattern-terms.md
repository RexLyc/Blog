---
title: "设计模式-术语"
date: 2022-01-22T14:17:41+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
软件工程中常有和设计模式相关的一些额外的术语，在这里进行一个整理。
<!--more-->
## 对象
1. O/RM（Object Relational Mapping，对象关系映射），将对象和关系型数据库进行绑定，以对象来表示关系数据。
    1. VO & PO，最基本的两种对象：
        1. VO（Value Object，值对象）：通常用于业务层之间（层内、各层之间）传递数据，可以和表对应，也可以不对应。由用户以new创建。
        1. PO（Persistent Object，持久对象）：向数据库添加或删除数据时产生的，仅存在于和数据库连接时，即仅存在于数据访问层。一定和表结构对应，需要实现序列化接口。
    1. DO（Domain Object，领域对象）：DDD（Domain-Driven Design，领域驱动设计）的术语，指抽象出来的业务实体。可以拥有含有业务逻辑的方法。
        - 和PO的主要区别：是否需要进行持久化。
    1. VO（View Object，显示层对象）：用于向前端输出的展示对象。
    1. TO（Transfer Object，传输对象）：用于在应用程序之间传输的对象。
    1. QO（Query Object）：查询对象，将所有支持用于查询的字段进行封装的对象。对应着SQL语句中的where子句。
    1. BO（Business Object，业务对象）：代表一个完整的业务流程，封装有业务逻辑的java对象，内部往往包含多个PO、VO（Value Object）进行业务操作。
        - 可以认为和DO等价。
    1. DAO（Data Access Object，数据访问对象）：夹在业务层和数据库资源中间，提供数据库的操作。例如Spring框架中的各种Mapper。
    1. DTO：（Data Transfer Object，数据传输对象）：用于前端和后端之间传输实际需要的内容的对象，一般只是一个表的部分字段。如果这个对象对应了界面的展示，那它就是VO（View Object）。
        - 但是：设计上并不推荐VO、DTO两者直接等同。因为数据应当可以和表现形式之间解耦（即相同的数据，也可以使用不同的方式进行展示）。
        - 和DO的区别：DO是完整的业务实体，但并不一定允许展示层查看所有字段、方法。
    1. POJO：符合Java Bean规范的简单对象。

## 函数式编程
参考文章[函数式编程合集]({{<relref "/content/post/language/functional-programming.md#基本概念">}})

## 代理
1. 代理的含义比较广泛，在不同场合中有不同的表现
1. 网络：
   1. 正向代理：客户端知道代理服务器的存在，也知道服务器的存在，但是由代理服务器完成请求。服务器不知道客户端的存在，只知道代理服务器。
   1. 反向代理：客户端不知道服务器的存在，只知道代理服务器的存在，由代理服务器调用具体的服务器完成请求。服务器知道客户端、代理服务器的存在。

## 池化技术
池化技术指的是提前准备一些资源，在需要时可以重复使用这些预先准备的资源。池化技术的优点主要有两个：提前准备和重复利用。常见的池：
- 线程池
- 连接池
- 内存池
- 对象池

### 线程池
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务。另一方面也可以控制任务的响应速度，避免等待线程创建和初始化阶段的延迟。如果线程数量超过了最大数量超出数量的线程，则任务需要排队等候，等其它线程执行完毕，再从队列中被取出执行。主要特点为：线程复用、控制最大并发数、管理线程。线程池的组成主要分为以下 4 个组成部分：
1. 线程池管理器：用于创建并管理线程池
2. 工作线程：线程池中的线程
3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行
4. 任务队列：用于存放待处理的任务，提供一种缓冲机制

比如Java 中的线程池是通过 Executor 框架实现的，该框架中用到了 Executor，Executors，ExecutorService，ThreadPoolExecutor ，Callable 和 Future、FutureTask 这几个类。其中ThreadPoolExecutor的构造方法形如
```java
/*
 * 默认线程数量（核心线程）、最大数量（允许创建一些非核心的临时线程）、空闲线程存活时间、存活时间单位、任务队列
 */
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, long keepAliveTime
    , TimeUnit unit, BlockingQueue workQueue)
{
    // 调用最基础的构造函数，还额外需要一个线程工厂、一个在任务过多而导致加入失败时的处理Handler
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit,
        workQueue, Executors.defaultThreadFactory(), defaultHandler);
}
```

Java线程池的工作流程
1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。就算队列里面有任务，线程池也不会马上执行它们。
2. 当再调用 execute() 方法添加一个任务时，线程池会做如下判断：
    1. 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；
    2. 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；
    3. 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
    4. 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常 RejectExecutionException。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到corePoolSize 的大小。

### 对象池
对于一些创建和销毁非常消耗资源，或者频繁进行创建销毁的对象，创建一个池子用来存储和复用是更好的选择。比如网络连接。其实线程池也是广义上的一种对象池。对象池的优缺点比较明显，在使用时需要根据场景来决定，不是所有的对象都适合池化。

优点：降低创建对象的消耗，提高响应速度
缺点：多线程情况下需要上锁，池大小难以确定

Java中比较常见的对象池就是Commons Pool。它在设计上有如下接口/规范
```java
// 池化对象需要实现的最基础接口
public interface PooledObjectFactory<T> {
    PooledObject<T> makeObject();
    void activateObject(PooledObject<T> obj);
    void passivateObject(PooledObject<T> obj);
    boolean validateObject(PooledObject<T> obj);
    void destroyObject(PooledObject<T> obj);
}

// 为了提高可用性，实际上更常用的类/接口是，分别提供了简化的接口，以及用key检索对象的能力
// public class BasePooledObjectFactory<>
// public interface KeyedPoolableObjectFactory<K,V>
```

从API设计上就可以看出，对象池的一种常见设计就是采用：借用-归还模式。其中
1. 借用：检查对象数量、不够则创建，超出池大小（maxActive）则阻塞知道有对象归还（或者报错）
2. 归还：恢复对象数量、有阻塞则唤醒阻塞，另外检测是否超出最大空闲（maxIdle），进行销毁

设计中的一些可用参数：maxActive、maxIdle、minIdle、maxWait（最大阻塞时间）、whenExhausedAction（对象池满时的策略）等等。

更多细节可以参考[基于Apache组件，分析对象池原理](https://cloud.tencent.com/developer/article/1984179)

### 内存池
内存管理不外乎三个层面：用户程序层，C运行时库层，内核层。一般所说的内存池都是用户程序层。内存池的情况相对复杂，C/C++在这方面做的优化比较多。内存池的出现原因更基础，优化对内存的申请和释放速度，对于频繁和大段内存的new/delete、malloc/free进行优化。在C运行时库层，这方面常见的对比有[ptmalloc、tcmalloc、jemalloc](https://www.cyningsun.com/07-07-2018/memory-allocator-contrasts.html)。也就是说实际上C/C++在库层面已经具备了一定程度上的内存池。

在C++的系列文章中也提到过SGI的[allocator内存分配器]({{<relref "/content/post/language/Cpp-difficulty.md#allocator空间配置器">}})。可以参考其中的实现。

不论是哪种内存池，其实所作的内容无非就是：申请大片内存、按照设定划分为存储不同大小对象的若干个链表、为内存申请分配链表上的空闲内存、恢复分配内存到空闲链表。

当然内存池技术还在不断发展，比如[知乎回答-论文HALO动态堆内存布局优化](https://zhuanlan.zhihu.com/p/687463484)。这里面提到了对内存池的一种优化是动态调整布局，对于一些亲和性(affinity)比较强的数据（经常一起访问使用），将他们从不同大小块（比如原来分别在8B、32B、96B的内存块中）的分配位置，移动到一起。

## 类型
1. 泛型
    1. [变型（variance）](https://zh.wikipedia.org/zh-cn/%E5%8D%8F%E5%8F%98%E4%B8%8E%E9%80%86%E5%8F%98)：描述具有父/子型别关系的多个型别通过型别构造器、构造出的多个复杂型别之间是否有父/子型别关系的用语。
    2. 协变
    3. 逆变
    4. 不变