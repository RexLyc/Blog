---
title: "Linux：Web和I/O系列书籍读书笔记"
date: 2024-03-26T15:56:21+08:00
categories:
- 计算机科学与技术
- Linux
tags:
- web
- io
- 施工中
- 工程
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/linux-web-io-series.png
---
本读书笔记记录和Linux环境下，Web开发，I/O模型相关的所有书籍。
<!--more-->
## 参考书籍列表
| 书籍名称 | C++版本 | 重点 |
| --- | --- | --- |
|Linux多线程服务端编程：使用muduo C++网络库 | 早于C++11 | 线程安全 |
| Linux高性能服务器编程 |  | TCP/IP和IO模型 |
| C++服务器开发精髓 |  |  |

## 术语
1. RTT：Round-Trip Time 往返时延。数据包发送和接收ACK确认的时间差。
2. RTO：Retransmission Timeout 超时重传时间。

## 工具和指令
在网络程序开发过程中，有一些指令和工具很有帮助，这里列举一下
| 名称 | 类型 | 作用 | 常用方法 |
| --- | --- | --- | --- |
| wireshark | 工具软件 | 抓包 | 图形化方式 |
| tcpdump | 指令 | 抓包工具 | 对不同协议、网卡、地址等进行筛选的抓包 |
| ifstat | 指令 | 网络流量检测 | 主要用于对流量的总体统计，如速率，总量 |
| lsof | 指令（list of open file） | 列出当前系统打开的文件描述符 | 一样也可以指定协议、地址、服务名等等 |
| nc | 指令（netcat） | 作为服务器或者客户端，构建网络连接 | -l -C等 |
| strace | 指令 | 跟踪程序运行过程中的系统调用和接收到的信号 | 相比于gdb调试，strace能更直观的看到运行时的一些信息 |
| netstat | 指令 | 查看网络信息统计 | -ntlp 显示地址，仅tcp在监听套接字，显示pid， |
| vmstat | 指令 | 查看进程信息、内存使用、CPU使用、I/O使用 | 设置采样频率和内容，磁盘和CPU也可以分别改用iostat/mpstat |
| route | 指令 | 查看路由表 | Flags注意，U活跃、G网关、H主机、DM重定向 |

## 协议栈快速复习
### IP
以下截取自维基百科：[ipv4](https://zh.wikipedia.org/wiki/IPv4)、[ipv6](https://zh.wikipedia.org/wiki/IPv6)
![ipv4协议](/images/book/linux-web-series/ipv4-protocol.png)
![ipv6协议](/images/book/linux-web-series/ipv6_header.svg.png)

作为网络层协议，IP协议提供无连接、不可靠、无状态的服务。它的核心任务是进行数据报的路由，即决定将数据发送的目标机器所需的路径。下图可以从右向左去看。
![ip模块的工作流程](/images/book/linux-web-series/ip-module-workflow.png)

IP数据报中比较重要的一点就是：在传输过程中，其源IP和目的IP，在转发的过程中一般保持不变。变动的是源MAC、和目标MAC地址。这也说明了IP协议是建立在数据链路层之上的协议。因此其实际在发送时，底层协议是按MAC地址进行发送的。

对于一台机器来说，接收到IP数据报之后，根据数据报中选择的路由机制，为其进行路由选择。同时也考虑是否是到本机，以及本机是否允许转发IP数据报。对于一个路由来说，允许数据报转发，它需要做一系列事情：
1. 检查TTL（TimeToLive，存活时间），如果已经是0，则丢弃
2. 检查源路由选择（IP协议可以基于源地址，选择自己的路由策略）。并根据该选项的需要，反馈ICMP报文，通知选站失败或者重定向。
3. TTL值减一，做其他的IP头部选项处理
4. 如果有必要，进行IP分片
5. 转发

而IPV6作为下一代网络层协议，提供了很多扩展。最重要的是增加了多播和流的功能，以及提供了更简洁的固定头部，以及可选的扩展头部。

> IPv6协议并不是IPv4 协议的简单扩展，而是完全独立的协议。用以太网封装的IPv6数据报和IPv4数据报，在以太网帧格式中，具有不同的类型值。


### TCP
![TCP协议格式](/images/book/linux-web-series/tcp-protocol.png)

上图是TCP协议格式，其中值得注意的是选项中的字段。这个字段的格式都是：kind、长度、内容。其中长度和内容根据选项而定。对这7种情况有一个印象即可：选项表结束标记、空操作、最大报文段选项、（滑动窗口）窗口扩大因子、选择性确认（用于SACK技术，提高重传性能）、时间戳选项（用于回路时间RTT计算）

带外数据（Out of band），指那些需要迅速通知对端的数据。TCP通过紧急URG状态和紧急指针来完成这种要求。

TCP最重要的特性之一就是提供可靠服务。而这一点是通过超时重传机制做到的。**每一个**报文段，都有一个定时器，用来查看是否被ACK确认。每一次重传的间隔时间都加倍（和超时重连一样）。在重试失败若干次后，由更底层的网络层协议接管（尝试重新查询网络IP等），直到放弃连接。重试次数和放弃连接前的尝试，都是可以配置的。Linux内核在网络协议栈上提供了相当多的可配置内容。

此外TCP还要考虑拥塞控制：慢启动（指数1/2/4/8增长）、拥塞避免（线性）、快重传/快恢复（连续三个重复ACK）。拥塞控制算法有多种实现。这些算法主要控制/使用了一些变量。
1. SWND发送窗口：收到第一个确认之前，可以一次连续写入网络的数据量。用于平衡网络延迟和网络拥塞
2. RWND接收窗口：接收方的可用于接收的窗口长度
3. CWND拥塞窗口：真正用于控制发送速率的变量。是通过前一轮的CWND和SWND，在每一轮重新计算。如果发生快重传快恢复，则减少到ssthresh
4. SMSS：发送端最大报文长度（未确认的全部数据长度），一般等于MSS
5. MSS：最大报文长度
6. ssthresh：慢启动阈值

### HTTP协议
参考[http协议]({{<relref "/content/post/Web/fullstack-protocol.md#HTTP">}})

考虑正向/反向代理的情况下，一次HTTP的连接过程。

1. 检查本地host文件，检查本地resolv.conf中的DNS服务器配置
2. 如果需要DNS，由代理服务器访问DNS服务器，查询目标域名IP（这个过程中会发送UDP数据报、并且可能需要再先发送ARP，查询配置的路由的MAC地址）
3. 代理服务器查询路由路径（MAC、IP）
4. http客户端，发送请求到代理服务器。（建立TCP连接、HTTP连接）
5. 代理服务器，发送请求到HTTP服务器。（建立TCP连接、HTTP连接）

> HTTPS主要是在传输层和应用层之间引入了一个安全套接字层（TLS/SSL），其对HTTP来说是透明的，应用将数据输入到安全套接字管道中，安全套接字层进行加密，并输出到传输层。

## 其他背景知识
1. 字节序：大（尾）端（Big Endian，高位字节存储在低位，更好记得就是尾部字节在高位）、小端反之
    ```cpp
    int a = 0x12345678;
    if(*(char*)(&a) == 0x12) {
        // 大尾端
    } else {
        // 小尾端
    }
    ```
    字节序不仅和平台有关，有时也和语言相关
    | 平台 | 字节序 |
    | --- | --- |
    | 网络传输 | 大端 |
    | x86 CPU（INTEL） | 小端 |
    | Moto CPU 摩托罗拉 | 大端 |
    | Arm CPU | 小端 |
    | Java虚拟机 | 大端 |
1. 

## socket编程
是Linux的网络编程的抽象。由于从传输层及往下，都是由内核实现的。因此内核向上提供的系统调用，即socket编程，需要提供一组接口，满足
1. 将应用程序数据从用户缓冲区中复制到 TCP/UDP内核发送缓冲区，以交付内核来发送数据，或者是从内核TCP/UDP接收缓冲区中复制数据到用户缓冲区，以读取数据
2. 应用程序可以通过它们来修改内核中各层协议的某些头部信息或其他数据结构，从而精细地控制底层通信的行为。
3. 不仅可以访问TCP/IP协议栈，也可以访问其他网络协议栈。例如UNIX本地域协议栈。

```cpp
// 摘抄，一个最基本的TCP Server例子
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <signal.h>
#include <unistd.h>
#include <stdlib.h>
#include <assert.h>
#include <stdio.h>
#include <string.h>

using namespace std;

static bool stop=false;
/*SIGTERM信号的处理函数,触发时结束主程序中的循环*/
static void handle_term(int sig ){
    stop =true;
}
int main( int argc,char* argv[]){
	cout << "hello world!" <<endl;
    signal(SIGTERM,handle_term);
    if(argc<= 3) {
        printf( "usage:%s ip_address port_number backlog\n", basename(argv[0]));
        return 1;
    }
    const char *ip=argv[1];
    int port = atoi( argv[2]);
    int backlog=atoi(argv[3]);
    int sock=socket(PF_INET,SOCK_STREAM,0);
    assert(sock>=0);
    /*创建一个IPy4 socket抢址*/
    struct sockaddr_in address;
    bzero(&address,sizeof(address));
    address.sin_family = AF_INET;
    inet_pton(AF_INET,ip,&address.sin_addr);
    address.sin_port = htons(port);
    int ret=bind(sock,(struct sockaddr*)&address, sizeof( address ));
    assert(ret!=-1);
    ret=listen(sock,backlog);
    assert(ret!=-1);
    /*循环等待连接,直到有SIGTERM信号将它中断*/
    while(!stop )
        sleep(1);
    /*关闭socket,见后文*/
    close( sock);
    return 0;
}
```

### 细节要点
1. socket编程的运行时效果，受到程序内运行时参数，和内核参数的共同影响。而二者的最终影响效果，受内核版本影响。在不同版本有不同的表现。例如
   1. Linux2.2之前和之后，backlog值对半连接和全连接队列的控制含义不同。而且在更高版本中（版本不详，不早于3.10，不晚于4.4），超出backlog的将不再建立半连接（因为建立了也无法accept）。这时可以通过netstat观察到连接队列溢出（SYNC_RECV状态）的情况。参考[backlog参数对TCP连接建立的影响](https://switch-router.gitee.io/blog/TCP-Backlog/)。

## 实践项目
在学习过程中所编写的项目代码，存储于[linux-web-concurrency-learn](https://github.com/RexLyc/linux-web-concurrency-learn)。

## 工具库、数据结构函数
1. 基本网络库
    | API/数据结构 | 功能 | 备注 |
    | --- | --- | --- |
    | sockaddr_storage | 通用，存储协议族、协议所用地址 | 内存对齐 |
    | sockaddr_un | 存储unix套接字协议，地址 |  |
    | sockaddr_in / sockaddr_in6 | 存储ipv4/ipv6协议，地址，端口，标记 |  |
    | in_addr / in6_addr | 存储ipv4/ipv6地址 |  |
    | getpeername / getsockname | 获取对端、本端地址信息 |  |
    | htonl() / ntohl() / ... | 网络字节序到主机字节序的各种数据类型转换 |  |
    | inet_addr() / inet_aton() / inet_ntoa() / inet_ntop() / inet_pton() | 字符串地址到ipv4/ipv6地址转换 | 有个别是不可重入函数，其返回的char*是一个内部静态变量，如有需要必须深拷贝 |
    | socket | 指定服务协议族、服务、子协议，创建套接字 |  |
    | bind | 绑定地址 |  |
    | listen | 开始监听 | 创建监听队列，如tcp有半连接队列和全连接队列 |
    | connect | 发起连接 |  |
    | close / shutdown  | 关闭连接 | close是将文件描述符的引用计数减一（计数归零会真正关闭）、shutdown则是直接强制关闭 |
    | recv / send | 发送、接受tcp流数据 | 通过flag控制发送、接收细节，例如发送和接收带外数据（URG） |
    | recvfrom / sendto | 发送、接收数据（TCP、UDP都可以） |  |
    | recvmsg / sendmsg | 发送、接收数据（TCP、UDP都可以） |  |
    | sockatmask | 查询当前套接字是否有带外标记的数据 | 需要结合recv，以及MSG_OOB标志，读取带外数据 |
    | getsockopt / setsockopt | 查询、设置socket选项 | 比如修改缓冲区大小、设置允许TIME_WAIT地址重用等 |
    | gethostbyname / gethostbyaddr / getservbyname / getservbyport / getaddrinfo / getnameinfo | 查询dns、根据名称（比如telnet）查询服务的基本信息（端口等） | 不可重入 |

1. 其他基本Linux的API
    | API | 

## 参考
- [[译] Linux 异步 I/O 框架 io_uring：基本原理、程序示例与性能压测（2020）](https://arthurchiao.art/blog/intro-to-io-uring-zh/)
- [小林coding 4.2 TCP 重传、滑动窗口、流量控制、拥塞控制](https://xiaolincoding.com/network/3_tcp/tcp_feature.html#%E9%87%8D%E4%BC%A0%E6%9C%BA%E5%88%B6)