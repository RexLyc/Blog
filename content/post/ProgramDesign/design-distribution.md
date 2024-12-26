---
title: "设计模式-分布式系统篇"
date: 2023-02-19T22:51:52+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
分布式系统是大型系统最终的归宿，不一定会写，但也要了解。
<!--more-->
## 关键词
1. 分布式系统：是一个硬件或者软件分布在不同的网络计算机上，彼此之间仅仅通过网络通信进行协调的系统。
    - 特征：
        1. 分布性
        1. 对等性
        1. 并发性
        1. 缺乏全球时钟，很难定义两个事件究竟是谁先谁后
        1. 故障总是会发生
    -  问题：
        1. 通信问题
        2. 网络分区（当网络发生异常时候，只有部分节点进行正常通信，另一些节点不可用，网络分区严重情况下面会导致“脑裂”）
        3. 三态（成功、失败、超时）
        4. 节点故障
1. 分布式事务：指事物参与者、支持事物的服务器、资源服务器以及事物管理器分别于分布式系统的不同节点之上。通常一个分布式事物中会涉及对多个数据源或者业务系统的操作。一般来说有7种常见的分布式事务实现方式。
    1. 2PC：两阶段提交，协调者发送事务给所有参与者，待所有人确认可以完成后，再发送事务提交确认。分为投票阶段、决定阶段。
    1. 3PC：三阶段提交，CanCommit（询问是否可以参与事务）、PreCommit（协调者发送请求，参与者开始执行，但保留Undo和Redo日志）、DoCommit（所有PreCommit成功，则意味着事务成功，否则abort中止事务，各个参与者进行Undo）
    1. TCC （Try-Confirm-Cancel）补偿模式：Try阶段尝试做事务，如果成功进行confirm，失败进行cancel。由于每个参与者都知道自己的Confirm、Cancel动作，因此协调者不需要单点，可以由多点业务程序负责。另外TCC引入超时机制，超时后会进行补偿。执行过程中也不会锁定整个分布式资源，而是以更小的资源粒度进行。
    1. 本地消息表：事务协调者维护一个事务消息表（本地数据库），用来记录对于不同参与者的业务处理是否成功，如果失败则需要发送通知给所有参与者回滚，重试直到成功
    1. 消息事务：使用消息队列中间件进行解耦，事务协调这发送消息到中间件，成功则提交，否则回滚，参与者作为消息消费者，重试直到事务结束
    1. 最大努力通知：一般也是使用消息队列进行事务通知，但是下游尝试几次之后如果仍然失败就可以放弃
    1. Sagas事务模型：其核心思想是将长事务拆分为多个本地短事务，由Saga事务协调器协调，如果正常结束那就正常完成，如果某个步骤失败，则根据相反顺序一次调用补偿操作。有事务协调器（TC）、事务（边界）管理者（TM）、资源管理器（RM）三个主要部分。
1. 一致性模型
    - 强一致性：（两段提交和三段提交模型, Paxos或者Raft算法）
    - 弱一致性：
    - CAP定理:在一个分布式系统（指互相连接并共享数据的节点的集合）中，当涉及读写操作时，只能保证一致性（Consistence）、可用性（Availability）、分区容错性（Partition Tolerance）三者中的两个，另外一个必须被牺牲。具体的，
        - 一致性：指数据在多个副本之间是否能够保持一致性的特性。在一致性的需求下，当一个系统在数据一致的状态下执行了更新操作后，应该保证系统的数据仍然处于一致的状态。
        - 可用性：指系统提供的服务必须一致处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内
        - 分区容错性：即分布式系统在遇到任何网络分区故障的时候，仍热能够保证对外提供满足一致性或可用性的服务
        > 在实际工程中，网络分区是非常可能发生的事情，因此分区容错性是几乎必须支持的，取舍一般也只能在一致性和可用性上
    - BASE理论：基本可用（Basically Available）、软状态（ Soft State）、最终一致性（ Eventual Consistency）三个短语的简写。
        - 基本可用：指分布式系统在出现不可预知的故障时候，允许损失部分可用性，保证核心服务可用。如响应时间上的损失（正常0.5ms之内的故障时候响应延时为1-2秒了）和功能上的损失（秒杀时候部分用户进行降级服务）
        - 软状态：指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性
        - 最终一致性：系统中所有的数据副本，在进过一段时间的同步后，最终能够达到一个一致的状态。
1. 其他相关术语：
    1. Servless：强调的是一种架构思想和服务模型，让开发者无需关心基础设施（服务器等），而是专注到应用程序业务逻辑上。
    1. SOA
    1. 微服务
1. 分布式系统架构设计
1. 分布式锁
    - 实现方式：
        1. 基于数据库
        1. 基于缓存
        1. 基于zookeeper
1. 分布式定时器
    - Quartz

## 高可用模型
本章节记录一下在各种中间件、分布式系统中，一些常用的高可用设计思路，或者是部署方式。

### MySQL
参考[MySQL 中常见的几种高可用架构部署方案](https://www.cnblogs.com/ricklz/p/17335755.html)

1. 基础方案：MySQL Replication，主从同步，一主多从。主库产生binlog，同步给从库，从库解析并生成relay log（中继日志）。最大的问题是主从同步延迟（受限于网络、从库性能、复杂事务），不仅会带来数据不一致性，而且更大的问题是在主库宕机后，从库可能丢失数据，有一些解决方案：
    1. 提高网络性能、从库性能
    2. 强行读主库
    3. semi-sync，半同步复制（以及增强半同步复制）。相比于异步复制、全同步复制，半同步复制是一个性能和一致性的权衡结果。
主从同步默认是异步的，这也正是主从同步延迟产生的原因之一。而使用半同步复制，可以让主库在至少收到一个数据库回复同步ACK之后，再向客户端反馈数据提交成功（提交事务结束）。这样能保证主库宕机，从库也至少有一台机器拥有更新的数据。

1. 好一点的：MySQL Group Replication。组复制。主要来解决数据一致性问题。在组复制模式下，一个MySQL集群可以使用多主模式（所有都可读可写），也可以使用单主模式（仍只用一个为主）。并且每一次写事务，先在主库本地进行，之后将操作广播（广播内容是write set写集，而且广播是原子的，全体接受或不接受）到全体组员，需要有半数以上的组员同意接收（组员批准，未发生行冲突），才能提交本次事务（其他成员接收该远程事务即可）。如果出现事务冲突，则广播中靠前的事务提交，后面的回滚。

1. InnoDB Cluster 是官方提供的高可用方案,是 MySQL 的一种高可用性(HA)解决方案。包含MySQL Shell、MySQL Router、MySQL Server三种组件，分别对应接口服务，请求路由，数据存储三种功能。

1. InnoDB ClusterSet。在前者基础上，通过将主 InnoDB Cluster 与其在备用位置（例如不同数据中心）的一个或多个副本链接起来，为 InnoDB Cluster 部署提供容灾能力。缺点比较多：集群之间只支持异步复制、只有一个集群是可写的。ClusterSet优先考虑可用性而不是一致性，以最大限度地提高系统的容灾能力。如果用户无法容忍故障转移期间事务或数据丢失，则不能使用InnoDB ClusterSet作为系统的解决方案

1. InnoDB ReplicaSet 是 MySQL 团队在 2020 年推出的一款产品，用来帮助用户快速部署和管理主从复制，在数据库层仍然使用的是主从复制技术。并不是ClusterSet的升级，使用ReplicaSet，主要是为了更好的写性能。

1. Master High Availability Manager and Tools for MySQL，简称 MHA。一套优秀的作为 MySQL 高可用性环境下故障切换和主从提升的高可用软件。分为MHA Manager（管理节点）和MHA Node（数据节点）。MHA Manager 可以单独部署在一台独立的机器上管理多个 master-slave 集群，也可以部署在一台 slave节点上。MHA Node 运行在每台 MySQL 服务器上，MHA Manager 会定时探测集群中的 master 节点，当 master 出现故障时，它可以自动将最新数据的 slave 提升为新的 master，然后将所有其他的 slave 重新指向新的 master。并在这个过程中向其他节点查询必要的数据，以尽最大努力补充数据，保证数据一致性。

## 网络相关
### Netty设计分析
Netty是Java网络编程中无法绕开的一个核心库，这里对其核心设计的关键词做一个总结。
- 分发模式（Dispatch / Reactor）：事件分发器，和若干个处理线程
    - 单Reactor单线程
    - 单Reactor多线程：业务处理变为多线程
    - 主从Reactor多线程：1 + M + N模式，较少的连接接收线程（不一定为1），以及若干个处理连接收发数据的线程，以及若干个业务处理线程
    > 常出现的，和Reactor模式对应的是Proactor模式，Reactor内部的执行流程仍然是同步的（非阻塞同步网络模式），数据I/O由用户进程处理（同步），Proactor则做到了异步，由系统完成I/O操作，用户回调只关注业务处理即可。总之**Reactor基于待完成的I/O事件**，而**Proactor基于已完成的I/O事件**。
- Netty线程模型：以主从Reactor多线程为基础
    - 事件部分
        - NioEventLoop：内部通过Executor创建线程，循环从Selector中收取待处理的事件，以及TaskQueue中的任务
        - Executor：实际类型是```EventExecutor```，它继承自JDK中的基础接口Executor框架，可以执行一个Runnable对象实例。是EventLoop内部实际上的执行单元。
        - TaskQueue：一个可以在NioEventLoop执行中被处理的任务队列，这里可以添加一些用户自定义的异步任务。尤其是一些耗时的任务，应该放在TaskQueue，而非Handler中执行（Handler在执行期间对于所在线程是同步的）。还有ScheduedTaskQueue用于存放定时任务。
        - BossGroup & WorkerGroup：Boss负责接收连接，Worker负责控制数据收发。在BossGroup中的Selector收到连接后，会创建用于收发数据的Channel，并注册到WorkerGroup的Selector
        - Selector：用于判断Channel发生何种IO事件，并进行处理选择
        - SelectionKey：用于描述IO事件状态，当Selector可以查询到SelectionKey时，代表有IO事件需要处理
    - 业务部分
        - Channel：用于支持对Socket进行非阻塞读写，注册到Selector上。常用的有```SocketChannel```、```DatagramChannel```、```FileChannel```、```ServerSocketChannel```
        - ChannelFuture：Netty的事件处理是异步的，每个事件注册后，返回Future，可以对其绑定Listener进行处理
        - ChannelInitializer：通道初始化器，通道在初始化时需要对上下文进行设定，尤其是为pipeline添加所有需要的handler
        - Handler：业务处理的最小单元，在Pipeline内。在实际类型上表现就是```ChannelHandler```的派生类。具体可以分为Inbound、Outbound对应入站和出站消息的处理
        - Pipeline：业务处理流程，按内部顺序，依次调用Handler
        - ChannelHandlerContext：一个上下文类型，同时包含了Channel、Handler、Executor
        - Buffer：用于支持Channel读写的缓冲区，典型的类型是```ByteBuffer```
        - ServerBootStrap & BootStrap：服务器和客户端的启动类
        - Encoder & Decoder：本质上也是一种ChannelHandler，专门用于数据收发时的编解码。如果有需要，可以泛化XXXToXXXEncoder、XXXToXXXCodec。
- 流程图：TODO
<!-- 插入一个  -->

- 更多内容，移步[《Netty In Action》读书笔记]({{<relref "/content/post/book/netty-in-action.md">}})

- Netty的典型代码
```java
// 取自参考博客：多图详解 Netty
public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
      	// 创建 BossGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        // 创建 WorkerGroup
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        // 创建服务器启动类
        ServerBootstrap bootstrap = new ServerBootstrap();
        // 添加配置
        bootstrap.group(bossGroup, workerGroup) // 设置 BossGroup 和 ChildGroup
                .channel(NioServerSocketChannel.class) // 设置 Channel 具体类
                .option(ChannelOption.SO_BACKLOG, 128) // 设置连接队列
                .childOption(ChannelOption.SO_KEEPALIVE, Boolean.TRUE) // 设置开启保活机制
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                      	// 把自定义 Handler 添加到 pipeline
                        socketChannel.pipeline().addLast(new NettyServerHandler()); 
                    }
                });
        // 绑定端口号
        ChannelFuture channelFuture = bootstrap.bind(new InetSocketAddress(9999)).sync();
        System.out.println("服务器启动成功！");
        // 阻塞直到通道关闭
        channelFuture.channel().closeFuture().sync();
        // 优雅地关闭 BossGroup
        bossGroup.shutdownGracefully();
        // 优雅地关闭 WorkerGroup
        workerGroup.shutdownGracefully();
    }

}
```

## 参考
- [分布式基础之CAP和BASE理论](https://www.jianshu.com/p/46b90dfc7c90) </br>
- [七种分布式事务的解决方案，一次讲给你听！](https://cloud.tencent.com/developer/article/1806989) </br>
- [几种分布式锁的实现方式](https://juejin.cn/post/6844903863363829767) </br>
- [三分钟了解 Serverless 是什么](https://zhuanlan.zhihu.com/p/340882159) </br>
- [多图详解 Netty](https://anye3210.github.io/2021/08/22/%E5%A4%9A%E5%9B%BE%E8%AF%A6%E8%A7%A3-Netty/) </br>
- [《Scalable IO in Java》](https://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)、[《Scalable IO in Java》译文](https://www.cnblogs.com/dafanjoy/p/11217708.html) </br>
- [Netty | 工作流程图分析 & 核心组件说明 & 代码案例实践](https://juejin.cn/post/7017602386747195429)
- [MySQL组复制MGR（一）-- 技术概述](https://www.cnblogs.com/lijiaman/p/13374694.html)