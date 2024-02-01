---
title: "WebRTC音视频实时交互技术-合集"
date: 2023-12-01T22:11:52+08:00
categories:
- 计算机科学与技术
- WebRTC
tags:
- 合集
- Web
- WebRTC
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/webrtc.png
draft: true
---
从webrtc入手，了解一点多媒体相关的信息技术。
<!--more-->
## 概述
1. 一些术语：
   1. PCM（Pulse Code Modulation）：脉冲编码调制，音频的原始数据
   2. YUV：视频格式，相对底层的原始数据
   3. 音频编解码器：Opus、AAC、iLBC、G.711/G.722、Speex。
   4. 视频编解码器：H264、H265、AV1、VP8、VP9
   5. 技术路线：RTMP、HLS、HTTP-FLV、DASH
   6. 评价指标：延迟、服务质量
      1. 服务质量标准：分辨率、帧率、码率
      2. MOS平均意见值：相同分辨率下，MOS值越高，需要的码率也越高。评价为5（优秀）、1（很坏）
   7. 音频处理3A问题：AEC回音消除、AGC自动增益、ANC降噪
   8. SVC（Scaled Video Coding）：可分层视频编码。将视频流分割为不同分辨率、质量、真速度层。
   9. Simulcast：多播，一个客户端发送多条不同码率大小的流
   
   > 编解码器没有绝对的优劣，各自都可能有擅长的领域

   > 音视频领域的技术路线和需求场景有关，直播内容需要更强的大规模分发能力，而视频会议、视频课程则需要更强的实时互动性

## 架构
1. 数据流向：音视频采集设备、音视频编码设备、网络设备、音视频解码设备、音视频播放设备
2. 客户端架构：
   1. 接口层：Web API（浏览器端）、WebRTC C++ API（Native开发）
   2. 会话层
   3. 核心引擎层：
      1. 网络传输层：网络I/O多路复用、P2P、SRTP
      2. 音频引擎层：NetEQ、Opus/iLBC Codec、3A
      3. 视频引擎层：Jitter Buffer、VP8/VP9 Codec、图像增强
   4. 设备层：Capture、Render、Network

## 基本原理

一个1对1通信的基本架构-流程图如下
![WebRTC架构图1：1](/images/screenshot/webrtc/signal-1on1-arch-flow.png)

图中展示了一对一通信的一个简化的整体架构和流程。其中
1. 检测客户端上可用的音视频设备
2. 开始采集
3. 录制
4. 打开同信令服务器的通信
5. 创建```RTCPeerConnection```对象，绑定音视频数据
6. 获得当前客户端在NAT映射中外网端口/IP（通过STUN/TURN进行NAT穿透）
7. 通过信令服务器进行媒体协商、交换Candidate（Call主叫/Called被叫的地址端口等）。
8. 通过```RTCPeerConnection```建立连接

### 信令服务器
信令服务器提供通信双方在业务层上的管理、以及信息的交换。完成用户创建房间、进入/退出房间，交换双方IP和端口等功能。所谓信令，也就是包含这些管理事件的信息。信令服务器需要对信令进行顺序限制，这种限制称为时序关系。

信令也分为客户端发起和服务端发起两个种类，一个简单的信令设计如下
1. 客户端发起：join、leave、message，分别代表客户端加入、离开、发送消息（用于同步各种元信息）
2. 服务端发起：joined、left、full，代表确认客户端加入、确认离开、房间满员

信令服务器在通信双方发起真正连接之前，负责传输各种元数据。具体有两个最重要的部分：交换SDP、交换Candidate。

交换SDP，Session Description Protocol会话描述协议。对于WebRTC而言，这一步交换也叫做媒体协商。双方需要协定在通信过程中使用的所有多媒体信息。比如支持的加密协议、使用的编解码协议等等。WebRTC对SDP协议进行了一定程度的扩展。而且SDP中有些字段，对于WebRTC来说是没有用的，比如对于连接所用的协议端口，因为WebRTC有专门的ICE流程，所以这些端口都只是象征性的内容。
![webrtc中使用的sdp协议内容](/images/book/webrtc/webrtc-sdp-summary.png)
从这份协议内容中也可以看出，其实一次WebRTC的通信过程，需要的元信息就是这些SDP中需要交换的内容：会话、媒体、连接、安全性、服务质量（feedback）、扩展。

交换Candidate，在媒体协商通过之后。两者需要交换用于建立网络连接所需的信息。一个Candidate至少由一个三元组（协议、地址、端口）组成。对于同一台机器，往往会提供多个Candidate（比如有多个网卡、多个地址）供其他机器连接。双方将会从最高优先级到最低优先级依次尝试建立连接。

信令服务器的基本实现就是这样了。实际上，只要你能有办法达成这些接口，无论你使用什么网络协议都是可用的。通常情况下，还是会使用基于TCP等可靠连接的上层协议，如HTTP/HTTPS、WS/WSS。

### ICE实现
ICE交互式连接建立协议。上一节提供了一个信令服务器的基本要求。本小节将会具体展开其中的ICE流程。也就是在媒体协商之后，尝试建立连接的步骤。

正如上一章节所说，在一个会话中的双方，可以有多种通信连接方式。对于最常用的UDP协议，其[Candidate](https://datatracker.ietf.org/doc/html/rfc5245#section-4.1.1.1)至少有如下4种。在了解这四种之前，需要先弄明白NAT协议，以及STUN/TURN服务。下面按优先级从高到低列举对应的四种方式
1. host连接：即通过ip地址、域名等方式，能够用地址直连
2. srflx（服务器端路由反射）：该方法使用客户端请求STUN/TURN服务器时，STUN服务器观测到的，客户端在NAT外网侧的地址
3. prflx（对端路由反射）：在ICE过程中逐渐发现的，体现了实际流量在网络路径中的一种变化。
4. relay（中继）：TURN服务器提供的中继地址

关于srflx和prflx。它们都是利用STUN协议进行的绑定，前者使用STUN服务器观测到的外网地址，后者是在具体的打洞过程中，逐渐发现的。以下内容来自GPT-4。
```markdown
1. 初始交换: WebRTC的两个端点通过信令服务器交换它们的ICE候选，这包括主机（host）、服务器反射型（srflx）、和中继（relay）。这一步还没有涉及prflx候选。

2. 连接检测开始: 每个端点使用收集到的候选信息开始进行STUN绑定请求，尝试直接与对方建立连接。这个过程中，每个端点不仅仅尝试发送数据，也在监听接收数据。

3. 发现prflx候选: 当一个端点通过其非prflx候选（例如host或srflx）向对方发送STUN请求时，如果通过的路径由于网络配置（如NAT设备）而被修改，对方收到的请求来源IP地址/端口与已知的任何候选不匹配。这时，接收端可以识别出这个新的源IP地址/端口对，这实际上反映了发送方的流经NAT后的外部（公共）IP地址/端口信息。接收端将这个信息作为prflx候选添加到自己的候选列表中，并可以通过STUN响应将其发送回原请求方。

4. 使用prflx候选: 一旦被发现并通过独立的连接检测确认是可用的，这个prflx候选就可以被使用来建立或维持端到端的连接。这种候选通常表明路径的某个地方有NAT设备对流量进行了修改。

在这个过程中，prflx候选的“计算”实际上是通过网络实际的响应和交互动态发现的，而非通过某种固定算法计算得到。这与host、srflx、relay这些更静态、预先可知的候选类型形成了对比，后者可以通过直接交互或配置得到。
```

### RTP和RTCP
WebRTC的核心传输协就是RTP和RTCP。在追求实时性的流式传输场景中，UDP要比TCP更优秀。因此主要考虑的也是基于UDP的应用场景。

![RTP协议数据包结构示意](/images/book/webrtc/rtp-protocol.png)
一些对于音视频数据流很重要的RTP头部字段：序号（判断乱序和丢包）、PT&SSRC（标记载荷数据类型、轨、数据源）、版本、Padding填充标记、CSRC（数据具体组成，比如一个数据由三个音频轨混音）、M标记（Marker标记一帧数据结束）、其他扩展字段（X标记、扩展字段类型、扩展字段内容）。

RTP协议簇中另一个协议就是RTCP，他和RTP都是应用层的协议。RTCP会对RTP进行控制，RTCP支持的消息类型很多，下面列举一些常见的类型
| PT载荷标识 | 缩写 | 全称 |
| --- | --- | --- |
| 200 | SR | Sender Report，发送端报告，包含一段时间内的发包数量等数据 |
| 201 | RR | Receiver Report，接收端报告，包含一段时间内的丢包量、丢包率、延时 |
| 202 | SDES | Source Description Packet，描述音视频媒体源 |
| 203 | BYE | Goodby Packet，标记媒体源下线 |
| 204 | APP | Application-defined，预留的应用层可解析内容 |
| 205 | RTPFB | Generic RTP Feedback，RTP传输层面反馈报文 |
| 206 | PSFB | Payload-specific Feedback，RTP上层业务负载层面的反馈报文 |

![RTCP协议数据包结构示意](/images/book/webrtc/rtcp-protocol.png)
RTCP的头字段相对简单，主要是版本、Padding填充标记、PT载荷、长度。

RTCP进行反馈最重要的是RTPFB、PSFB这两种类型的报文，其数据部分会再填充不同类型的子报文，以实现对发送方的反馈。
1. RTPFB：NACK（常规丢包反馈）、TMMBR/TMMBN（废弃，和码流有关）、TFB（TCC算法反馈报文，用于为发送端计算下行带宽）
2. PSFB：PLI（无法解码的接收端，请求一帧关键帧）、FIR（新加入会话的接收端，请求一帧关键帧）、REMB（接收端的带宽评估反馈）

### 流和轨
多媒体内容被抽象为MediaStream和MediaStreamTrack。一个MediaStreamTrack是一个单独多媒体数据种类，比如视频、音频。一个MediaStream则是若干个需要进行事件同步的MediaStreamTrack。

## 源码指南
1. RtpPacket：封装对RTP协议的读写

## 实战内容
一个支持RTSP接入的服务器端、以及前端Demo

## 其他内容
### 术语
1. NAT：网络地址转换协议。在出口路由器上，将内网IP、端口，映射为出口的外网IP、端口。[NAT的实现](https://info.support.huawei.com/info-finder/encyclopedia/zh/NAT.html)中也有不同的策略和分类。
   1. 从STUN协议中对NAT的分类：完全锥形、限制锥形、端口限制锥形、对称型NAT。这些分类的区分方式，是判断外网程序能否通过一个经过NAT映射后的外网IP+外网端口，去访问内网主机。
1. STUN：Session Traversal Utilities for NAT，[NAT会话穿越实用工具](https://info.support.huawei.com/info-finder/encyclopedia/zh/STUN.html)。STUN协议为C/S架构，服务器为客户端提供自身和对端的NAT信息。分为两个步骤，探测和打洞。
   1. 探测阶段：算法示意可见[Wiki介绍](https://zh.wikipedia.org/wiki/STUN)。无法穿透的可能主要有三种：不支持UDP、防火墙、对称型NAT。在这一步客户端能得知自己是否位于NAT设备后，以及对端的地址。
   2. 打洞阶段：客户端向对端的地址持续发送两种报文（NAT前地址、NAT后地址），直到联通（发送报文，并等待双方的NAT设备都建立表项）。注意如果在同一个内网，则使用NAT前地址就可以联通。
      > 双对称型很难打洞，但如果只有一个对称型，仍然可以通过控制打洞顺序来完成NAT穿透

2. TURN：Traversal Using Relay NAT，中继穿越NAT。当STUN无法满足时（比如对称型NAT），可能需要使用TURN。这是一种中继穿越方式，需要公网的TURN服务器具备足够的带宽。TURN也依赖STUN协议，它有两种发送数据的方式。
   1. Send/Data indication指令：用XOR-PEER-ADDRESS/DATA属性，在每一次传输都指明数据，以及目标Peer的地址。
   2. ChannelBind指令：在创建连接时就制定好参与一个通道的Peer，此后每次发送的时候指明Channel。
   > 注1：注意区分TURN协议中的TURN Client和Peer。Client是发起一次TURN协议Allocation请求的客户端，其他参与者称为Peer。当然也可以让通信双方都是TURN Client。TURN服务器会为TURN Client准备一个Relay地址，任何发送到Relay地址的数据，都会被转发给对应的TURN Client。
   > 注2：TURN Client向TURN Server发送数据时，用XOR_PEER_ADDRESS、Channel等方式，决定数据发送给哪个Peer。

3. SDP内容中常见的协议栈：UDP/TLS/RTP/SAVPF
   1. 协议最底层是UDP
   2. 在UDP之上，用DTLS进行传输层加密
   3. 再之上运行RTP，以及RTCP
   4. 最上层使用SAVPF进行反馈控制

### 常见问题和方法
1. 减少数据量：压缩、SVC技术、Simulcast技术、动态码率、甩帧
2. 适当增加时延：缓冲队列
3. 提高网络质量：降低丢包、延迟、抖动。NACK/RTX、FEC前向纠错、JitterBufer、NetEQ、拥塞控制
4. 快速评估带宽：Goog-REMB、Goog-TCC、NADA、SCReAM

### 关注点
1. ORTC对SDP的替代，以及由此带来的RTC开发风格的变化。

## 参考
1. 《WebRTC音视频实时互动技术》
2. [P2P技术原理浅析](https://keenjin.github.io/2021/04/p2p/)