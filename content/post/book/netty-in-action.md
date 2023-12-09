---
title: "《Netty In Action》读书笔记"
date: 2023-12-04T22:14:02+08:00
categories:
- 读书笔记
- 技术书
tags:
- 施工中
- Netty
- 框架
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/netty.png
draft: true

---
Netty作为一个广泛应用的Java高性能网络框架，不仅可以作为框架使用，其本身的设计模式和思路也很值得学习。
<!--more-->

> 本章绝大多数内容均拷贝自原书，根据Netty、Java版本有一定变动。

## 基本架构
1. 基本术语：
   1. Channel：在Netty中代表了到一个通信实体的连接，可以对连接进行打开、关闭，并对活跃连接上进行读写，
   2. 回调：Netty使用回调来处理各类事件，使用回调时，须泛化实现对应接口的事件回调函数
   3. Future：Netty提供了自己的实现```ChannelFuture```，可以通过对其添加监听器```ChannelFutureListener```，作为另一种在事件结束时通知应用程序的方式
   4. 事件和ChannelHandler：Netty提供的事件包括但不限于
        - 连接开关
        - 记录日志
        - 数据转换
        - 流控制
        - 用户事件
        </br>这些事件将会根据其数据发送方向，分别在ChannelHandler中由预设、或用户实现的各种事件处理器进行处理，并支持对事件转发，达到链式处理的效果。
2. 基本原理：
    - Netty基于NIO，整体原理就是由Selector监听I/O事件，并派发给对应的处理线程
    - Netty为每一个Channel创建一个事件循环EventLoop
    - EventLoop在执行期间注册回调，并将事件派发
3. 重点类型：
   1. 各类handler：```SimpleChannelInboundHandler```、```ChannelHanderAdapter```，区别主要在于业务逻辑对消息的处理，以及对资源的处理（是否释放消息所用内存）。如果使用不当，可能会抛出资源无法释放等异常提示。
   2. ```ChannelHandlerContext```：同时持有关联的```Channel```，```Handler```，```Pipeline```。负责在```Handler```链上传递出站入站消息，向对端发送数据等功能。
4. 样板代码：改动自原书，版本基于Netty-all 5.0.0.Alpha2
```java
// 回显服务端ChannelHandler
// 书中ChannelInBoundHandlerAdapter已废弃
// Sharable说明该Handler可供多个channel共享使用
@ChannelHandler.Sharable
public class MyServerChannelHandler extends ChannelHandlerAdapter {

    // 从通道中读取到数据的回调
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf) msg;
        System.out.println("get msg: " + in.toString());
        // ChannelHandlerContext同时关联channel、handler、pipeline
        // 将输入数据回显给对端。这里是写入缓冲，不一定发送
        ctx.write(in);
    }

    // 读取完成回调，此时代表当前批次读取没有更多的数据了（读取长度为0）
    // 对于流式的数据，并不是绝对的正确
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        // 将发送缓冲区刷新
        // writeAndFlush返回一个ChannelFuture。这里在刷新完成后，关闭channel
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }

    // 异常处理
    // 正常情况下，各个Handler链式处理消息或者异常，因此应当至少有一个exceptionCaught实现
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}

// 服务器启动代码
public class ServerClass {

    public final int port ;

    public ServerClass(int port) {
        this.port = port;
    }

    public static void main(String args[]) throws Exception {
        int port = 5000;
        new ServerClass(port).start();
    }

    public void start() throws Exception {
        // 自定义的ChannelHandler
        final MyServerChannelHandler serverHandler = new MyServerChannelHandler();
        // 创建核心EventLoopGroup
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            // 创建服务器
            ServerBootstrap b = new ServerBootstrap();
            // 配置EventLoopGroup，这里设置成了分享，实际也可以设置BossGroup和WorkerGroup不同
            b.group(group)
                // 配置Channel类型
                .channel(NioServerSocketChannel.class)
                // 服务器本地地址、端口
                .localAddress(new InetSocketAddress(port))
                // 这里的child指子Channel
                // ChannelInitializer是一种特殊的ChannelHandler
                // 特化了Channel初始化回调
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch)
                            throws Exception {
                        // 设置了Sharable，可以在多个Channel间共享
                        ch.pipeline().addLast(serverHandler);
                    }
                });
            // 阻塞等待服务建立（绑定到端口）
            ChannelFuture f = b.bind().sync();
            // 阻塞等待服务终止
            f.channel().closeFuture().sync();
        } finally {
            // 释放资源，优雅退出
            group.shutdownGracefully().sync();
        }
    }
}


// 客户端Handler
// SimpleChannelInboundHandler接口有调整
@ChannelHandler.Sharable
public class MyClientChannelHandler extends SimpleChannelInboundHandler<ByteBuf> {

    // 通道可用回调，即连接建立
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", StandardCharsets.UTF_8));
    }

    // 若实现channelRead，其优先级高于messageReceive
    // channelRead0已经取消
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(
                "Client  channelRead received: " + ((ByteBuf)msg).toString(CharsetUtil.UTF_8));
    }

    // 唯一必须实现的接口
    @Override
    protected void messageReceived(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
        System.out.println(
                "Client messageReceived received: " + byteBuf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}


// 客户端启动类
public class PeerClass {
    private final String host;
    private final int port;

    public PeerClass(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            // 普通启动类
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)
                    // 配置远程地址
                    .remoteAddress(new InetSocketAddress(host, port))
                    // 配置初始化ChannelHandler
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch)
                                throws Exception {
                            ch.pipeline().addLast(
                                    new MyClientChannelHandler());
                        }
                    });
            // 阻塞等待连接建立
            ChannelFuture f = b.connect().sync();
            // 阻塞等待连接关闭
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }
    public static void main(String[] args) throws Exception {
        new PeerClass("localhost", 5000).start();
    }
}

```
## 组件设计
1. 先思考：做一个高性能的IO框架都需要什么
   1. 数据结构：表达不同类型的连接、表达不同的异步事件
   2. 接口：统一的消息处理接口、统一的异常处理、统一的编解码
   3. 流程：允许消息在不同的处理器之间流转
   4. 通用功能：日志、多线程
### Channel系
1. ```Channel```：对不同类型连接的抽象
   1. 内置类型：NIO（NioSocketChannel/NioDatagramChannel）、OIO（阻塞流）、Local（一个JVM内）、Epoll（Linux下推荐）、Embedded（一种特殊的用于嵌入其他ChannelHandler中的Channel）
   2. 关联：Pipeline。当Channel创建时，自动就创建了Pipeline，用来容纳各种消息处理器。
   3. **线程安全**
2. ```ChannelHandler```：最常被继承的类型，业务核心一般就在相应的回调函数的实现中
   1. 子接口：
        - 入站：```ChannelInboundHandler```
        - 出站：```ChannelOutBoundHandler```
   2. Handler实现的功能包括但不限于：
      1. 数据格式转换
      2. 异常通知
      3. Channel活跃状态变化通知
      4. 用户自定义事件
      5. Channel到EventLoop是注册、注销通知
   3. 性能要求：不能在处理过程中使用同步阻塞接口（比如各类```ChannelFuture```的```sync```）
3. ChannelPipeline：作为Channelhandler链的容器
4. ChannelHandlerContext：当Handler添加到Pipeline中时，分配获得
   1. 意义：代表handler和ChannelPipeline之间的绑定关系。多用于写出站数据，该数据将从出站的尾端开始流动。
   > 虽然可以直接写入Channel，但会导致出战数据直接从下一个Handler开始流动。未复现出理解的效果。
5. ChannelConfig：支持热更新的配置

### 数据系
1. 设计思考：网络传输的最小单位基本就是字节，而Java提供的ByteBuffer过于底层，接口复杂，对于网络通信来说不够方便。对缓冲区的设计应当考虑：
   1. 性能：提高拷贝、移动效率、池化、控制GC
   2. 容量：支持动态变化
   3. 访问：随机访问、顺序访问、能支持查找
   4. 线程安全
2. ```ByteBuf```：Netty的字节缓冲区组件
   1. 特点：
      1. 支持缓冲区类型扩展
      2. 零拷贝支持
      3. 容量按需增长
      4. 读取、写入双索引（读写不需要切换，不需要做原生Buffer的flip操作）
      5. 支持链式调用、引用计数、池化
   2. 缓冲区模式：
      1. 堆缓冲区：创建并使用分配到堆中的数组，又称支撑数组。```ByteBuf.array()```，可以通过```hasArray```来判断当前```Bytebuf```是否使用支撑数组。
      2. 直接缓冲区：直接使用外部分配的内存（Native分配，不在JVM的内存管理范围内），因为减少了从Native内存拷贝到JVM堆的过程，往往能提升性能。```ByteBuf.getBytes```
      3. 复合缓冲区```CompositeByteBuf```：可由多个```ByteBuf```聚合而成。这在一些需要拼接数据内容的协议中十分实用，不同的```ByteBuf```往往由多个模块负责填充。最终聚合到一起，减少不必要的拷贝。示例代码
        ```java
        // 通过Unpooled创建非池化缓冲区
        CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
        ByteBuf headerBuf = ...; // 可以是堆缓冲，也可以是直接缓冲
        ByteBuf bodyBuf = ...;
        // 将已有Buf合并为复合缓冲区
        messageBuf.addComponents(headerBuf, bodyBuf);
        // 业务操作
        // ...
        // 支持对Buf再分离
        messageBuf.removeComponent(0);
        for (ByteBuf buf : messageBuf) {
            System.out.println(buf.toString());
        }
        // 或者通过直接缓冲区的访问方式访问
        int length = messageBuf.readableBytes();
        byte[] array = new byte[length];
        messageBuf.getBytes(messageBuf.readerIndex(), array);
        // 继续使用array
        // ...
        ```
    3. 其他功能
       1. 字节级读写：其中```get/set```不会调整读写索引，```read/write```会调整索引
       2. 派生缓冲区：通过```slice```、```duplicate```等方式建立一个视图，和原缓冲区共享同一个数据源，浅拷贝
       3. 复制：通过```ByteBuf.copy```、```Unpooled.copiedBuffer```，拷贝一个完全独立的新缓冲区，深拷贝
       4. 为了支持池化，提高GC性能，提供引用计数```refCnt```。
            > 需要考虑检测缓冲区泄露
3. ```ByteBufHold```：一个用于保管ByteBuf的接口类型。
   1. 作用：实现，并可以在其中保存其他属性字段，也用于支持池化等高级特性。
4. ```ByteBufAllocator```接口：分配器，分配```ByteBuf```，也是实现ByteBuf池化的接口
   1. 内置实现：```PooledByteByfAllocator```、```UnpooledByteBufAllocator```，顾名思义，前者池化，后者不池化。其中后者还提供了```Unpooled```静态工具类
   2. 和```Channel```关系
      1. 受ChannelConfig控制，也可以在启动服务器、客户端时修改分配器
      2. 可以从```ChannelHandlerContext.alloc()```获取到分配器实例
5. ```ByteBufUtil```：静态工具类
    - ```hexdump()```方法


### 其他
3. ```EventLoop```：
   1. 关联：```Channel```、```EventLoop```、```Thread```、```EventLoopGroup```
   2. 关联关系：
        - 一个Group包含多个Loop
        - 一个Loop在生命周期内只绑定一个Thread
        - 一个Channel在生命周期内只注册到一个Loop
        - 一个Loop可以拥有多个Channel


## 核心架构
### BossGroup

### WorkerGroup


## 线程模型

## 编解码器

## 网络协议

## 案例分析

## 参考资料
1. 《Netty In Action》