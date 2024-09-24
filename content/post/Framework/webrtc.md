---
title: "WebRTC音视频实时交互技术-合集"
date: 2023-12-01T22:11:52+08:00
categories:
- 计算机科学与技术
- WebRTC
tags:
- 合集
- Web
- 暂停更新
- WebRTC
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/webrtc.png

---
从webrtc入手，了解一点多媒体相关的信息技术。
<!--more-->

> 本书在详解部分，主要以Windows下的实现为主，因此在实际的线程模型、网络模型、多媒体设备API方面，不同系统会有所差异。但无伤大雅。

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
1. RTPFB：NACK（常规丢包反馈）、TMMBR/TMMBN（废弃，和码率有关）、TFB（TCC算法反馈报文，用于为发送端计算下行带宽）
2. PSFB：PLI（无法解码的接收端，请求一帧关键帧）、FIR（新加入会话的接收端，请求一帧关键帧）、REMB（接收端的带宽评估反馈）

### 拥塞控制
WebRTC被广泛应用的原因就是其具备非常优秀的服务质量。通过增加带宽、减少数据量、提高音视频质量、适当增加时延以及更准确的带宽评估等方法来提升音视频服务质量的。在这些方法中，减少数据量、适当增加时延和更准确的带宽评估被统称为拥塞控制。

WebRTC有多种拥塞控制算法可供选择，如：GCC（Google Congestion Control）、BBR（Bottleneck Bandwidth and Round-trip propagation time）、PCC（Performance-oriented Congestion Control）。其中GCC是最常用的算法。

![GCC的Transport-CC算法](/images/book/webrtc/transport-cc.png)
GCC：基于延时对网络状态进行评估，这种手段主要是为了防止发生网络拥塞。具体仍然有多种算法实现，如Goog-REMB、Transport-CC（上图）两种。前者是在接收端进行网络质量的估算，并反馈给发送端调整码率。后者是接收端简单的反馈网络质量数据（延时、丢包率等），在发送端直接统一进行网络质量估算和控制。评估的内容就是网络的**带宽**。
   > 网络质量估计的算法核心，分别是[卡尔曼滤波器](https://www.kalmanfilter.net/CN/background_cn.html)，TrendLine滤波器。

另外还有一种基于丢包的拥塞评估算法。在网络中确实出现大量丢包时进行使用。算法利用丢包率分类处理，低于2%认为网络质量较好，可以提高码率，2%~10%说明匹配，不用调整，大于10%说明网络质量差需要降低码率。

![webrtc的拥塞控制流程](/images/book/webrtc/webrtc-network-control.png)
如图所示是WebRTC的拥塞控制流程。系统在初始状态下，设定一个初始带宽（比如500kbps），并发送数据，等待RTCP数据报的反馈，并进一步评估带宽，控制编码器和Pacer，调整视频编码码率，以及发送时的码率。并依次循环工作。

对拥塞控制算法的性能评价主要从两方面进行：和其他连接并存时的性能（公平性，既不过分抢占，也不会发生“饥饿”），对网络带宽波动的响应情况。理想中的拥塞控制算法应该能做到：对网络带宽变化尽快响应（不考虑丢包），和其他网络流量公平共存（各种不同协议），在有丢包发生的网络环境中尽快评估带宽、调整码率并保持平稳。

最后整体的数据流图如下
![webrtc数据流](/images/book/webrtc/webrtc-dataflow.png)

### 流和轨
多媒体内容被抽象为MediaStream和MediaStreamTrack。一个MediaStreamTrack是一个单独多媒体数据种类，比如视频、音频。一个MediaStream则是若干个需要进行事件同步的MediaStreamTrack。

流的编解码由WebRTC内部处理，对于视频、音频，WebRTC都提供了多种编解码器进行使用。由上一节可知，编码器在实际运行时输出的码率受拥塞控制管理。

此外在流中还可能混合FEC编码。FEC即前向纠错（forward error correction）。通过在数据中添加FEC数据帧，在部分数据包丢失时，直接利用已接收数据包，和FEC数据帧，恢复丢失的数据包。

## 源码指南
### 环境搭建
由于WebRTC源码非常庞大，而且其在不同平台上的实现不尽相同，因此本节仅记录书上提到的部分内容。并未尝试实际搭建。

书中的环境推荐：Windows 10（10.0.1.19041+），Visual Studio 2019+，NTFS文件系统（需要大文件支持）

搭建步骤
1. 下载、编译WebRTC源码的工具集[depot_tools](https://storage.googleapis.com/chrome-infra/depot_tools.zip)
2. 执行指令
   ```sh
   mkdir webrtc -checkout
   cd webrtc -checkout
   # fetch gclient都是depot_tools的工具
   fetch --nohooks webrtc
   gclient sync

   cd src
   gn gen out/Default
   ninja -C out/Default
   ```
> 如果有网络问题，建议使用国内声网的镜像

目录阅读指南

主目录

| 目录 | 说明 |
| --- | --- |
| api | WebRTC接口层，为浏览器提供的接口 |
| audio | 音频引擎 |
| base_override | 编译时使用 |
| call | 呼叫逻辑接口层 |
| common_audio | 音频算法 |
| common_videosdk | 视频算法 |
| data | 存放那个音视频数据 |
| examples | WebRTC Demo |
| logging | 日志相关代码 |
| media | 媒体引擎层，会调用音视频引擎 |
| modules | 存放一些比较独立的模块 |
| p2p | 端到端网络 |
| pc | PeerConnection相关 |
| rtc_base | 存放一些基础代码 |
| rtc_tools | 存放一些服务质量相关的工具 |
| sdk | 存放移动端代码 |
| stats | 存放各种统计信息 |
| system_wrapper | 操作系统相关代码 |
| tools_webrtc | 存放一些WebRTC性能相关的工具 |
| video | 视频引擎 |

modules目录
| 目录 | 说明 |
| --- | --- |
| audio_coding | 音频编码 |
| audio_device | 音频设备 |
| audio_mixer | 混音相关 |
| audio_processing | 音频3A处理 |
| congestion_controller | 拥塞控制，TCC、BBR等 |
| desktop_capture | 桌面采集 |
| pacing | 平滑处理 |
| remote_bitrate_estimator | 远端带宽评估 |
| rtp_rtcp | 协议 |
| third_party | 第三方库 |
| utility | 线程相关工具 |
| video_capture | 视频采集 |
| video_coding | 视频编码 |
| video_processing | 视频特效处理 |
| bitrate_controller | 本地带宽评估，TCC |

### 核心设计
![webrtc线程](/images/book/webrtc/webrtc-thread-model.png)
线程模型：WebRTC使用多线程来完成各个部分的业务。WebRTC的线程有两个层次。
1. 第一层：网络线程、工作线程以及信令线程。信令线程可以直接使用主线程。
2. 第二层：网络、工作、信令线程会根据业务创建子线程。
   1. 工作线程创建的子线程：视频编码线程、解码线程、音频编码线程、Pacer线程
   2. 信令线程（主线程）创建的子线程：视频采集、视频渲染、音频采集、音频渲染
> 第二层线程是否存在，以及存在数量，都和业务相关。并不是绝对的。



![webrtc网络模块](/images/book/webrtc/network-module.png)
网络模型：WebRTC的网络模块主要有八个部分（还有一些非核心内容），和图中编号对应的他们分别是
1. 数据包发送时的调用栈（接收反之同理）：BaseChannel接收待发送数据、切换到网络线程并逐个类进行处理（组包、加密、发送）
2. 网络模块接口：收集本地Candidate、添加远端Candidate
3. 底层端口分配
4. 网络端口抽象
5. 创建抽象Socket：为UDP、TCP、STUN等协议提供统一的网络接口
6. 底层Socket封装
7. Socket工厂类
8. 异步I/O事件处理

而在网络连接的建立过程中，主要有2个步骤
1. 收集、上报Candidate：这个步骤又可以进一步细分为获取网卡、创建抽象Socket、创建UDPPort等、创建Candidate
   ```cpp
   // 收集Candidate的调用堆栈
   // 1 -> PeerConnection::SetLocalDescription()
   // 2 -> JespTransportController::MaybeStartGathering()
   // 3 -> P2PTransportChannel::MaybeStartGathering()
   // 4 -> BasicPortAllocatorSession::StartGetheringPorts()
   // 5 …
   // 6 -> BasicPortAllocatorSession::OnAllocate()
   // 7 -> BasicPortAllocatorSession::DoAllocate()

   void DoAllocate(bool disable_equivalent) {
      // 获取网卡
      std::vector<> networks = GetNetworks();
      for (uint32_t i = 0; i < networks.size(); ++i) {
         // 从网卡创建Socket
         AllocationSequence* sequence =
            new AllocationSequence(this , networks[i], /*...*/);
         // 从Socket创建UDPPort、StunPort对象，并检测是否有可用的外网映射
         sequence->Init();
         sequence->Start();
      }
   }

   // 上报Candidate时的调用堆栈
   // 在本地准备完成后
   // 1 -> UPPPort::OnLocalAddressReady()
   // 2 // 生成Candidate
   // 3 -> Port::AddAddress()
   // 4 -> Port::FinishAddingAddress()
   // 5 -> BasicPortAllocatorSession::OnCandidateReady()
   // 6 // 对于被呼叫方，如果此时已经有Remote Candidate,则生成Connection
   // 7 -> P2PTransportChannel::OnPortReady()
   // 8 // 生成Connection后，重新回到OnCandidateReady执行
   // 9 -> BasicPortAllocatorSession::OnCandidateReady()
   // 10 -> P2PTransportChannel::OnCandidateReady()
   // 11 -> JespTransportController::OnTransportCandidateGathered_n()
   // 13 -> PeerConnection::OnTransportCandidateGathered()
   // Ice是ICE消息
   // 14 -> PeerConnection::OnIceCandidate()
   // 15 -> PeerConnectionObserver::OnIceCandidate()
   // 16 -> ...进一步上报给信令系统
   ```
2. 创建连接：通信双方在建立连接的过程中，可能存在多个可用Connection。建立之后会进行排序和STUN协议的Ping测试，选出最优质的连接作为最终的通信信道。
   ```cpp
   // 添加Candidate，建立连接
   // 1 -> PeerConnection::AddIceCandidate()
   // 2 -> PeerConnection::UseCandidate()
   // 3 -> JespTransportController::AddRemoteCandidate()
   // 4 -> JespTransport::AddRemoteCandidate()
   // 5 -> P2PTransportChannel::AddRemoteCandidate()
   // 6 -> P2PTransportChannel::FinishAddingRemoteCandidate()
   // 7 -> P2PTransportChannel::CreateConnections()
   // 8 -> UDPPort::CreateConnection()
   ```



音视频采集和播放
![音频处理流程](/images/book/webrtc/webrtc-audio-process.png)
音频处理的主要模块包括：ADM、音频引擎。在WebRTC进行音频协商时，会使用ADM启动两个线程，分别用于采集音频数据、播放音频数据。
1. 音频采集线程：驱动设备按照采样参数采集数据，每一帧数据交付到AudioDeviceBufer处理，并由音频引擎对其进行重采样到目标规格，并做前处理（回音消除、降噪）。后续进行编码、平滑处理。
   > 音频前处理可能由硬件完成，在这种情况下，AP模块（AudioProcess）将会跳过软件处理步骤。
1. 音频播放线程：由网络线程将数据交给音频引擎，进行解码，解码后做混音，再写入AudioDeviceBufer，并最终向声卡缓冲区填充数据。

![视频处理流程](/images/book/webrtc/webrtc-video-process.png)
理解视频处理流程主要理解两点，一个是抽象概念VideoTrack、一个是视频数据的相关对象的创建以及流转顺序
1. VideoTrack：VideoTrack一头连接视频源（VideoTrackSource），一头连接编码器（VideoStreamEncoder）或本地预览窗口。
   ```cpp
   // VideoTrackSource，创建视频源
   // 创建过程中会根据采集设备，启动对应驱动，采集视频数据
   rtc::scoped_refptr<CapturerTrackSource>
      video_device = CapturerTrackSource::Create();

   // VideoTrack，创建轨
   rtc::scoped_refptr<webrtc::VideoTrackInterface>
      video_track_(
         peer_connection_factory_ ->CreateVideoTrack(
            kVideoLabel ,
            video_device
         );
      );
   // 设置目标，即视频轨的消费者
   main_wnd_ ->StartLocalRenderer(video_track_);
   ```
2. 视频数据流转：创建VideoTrack（内部创建VideoSource、VcmCapturer）、设置本地预览或远程发送（为VcmCapturer添加回调）、VideoCaptureModule获取视频数据、回调VcmCapturer、转交给VideoBroadcaster进行分发
   ![视频数据流转](/images/book/webrtc/webrtc-video-dataflow.png)

### 部分接口设计
1. 线程切换：在有了线程模型之后，就需要考虑数据在不同线程之间的流转方式。WebRTC将数据流转抽象为不同线程对一个任务消息的输入输出。内部使用消息队列来完成同步。具体的API有多种调用：Send/Post/Invoke/PostTask，分别支持同步和异步的不同用法。本段代码引用都**省略了同步锁**。

   对于Post/PostTask，以下是书中摘取的部分核心代码。PostTask是对Post的进一步封装，能够更方便的使用匿名Lambda函数完成OnMessage处理。
   ```cpp
   // 所有线程，都需要有数据流转能力
   // 只截取书中引用的核心代码，每个函数的实际具体实现都更加复杂
   class Thread {
      // 线程内的Post函数声明和核心实现
      // 记录了来源线程、消息处理器、消息类型ID、消息数据指针
      // time_sensitive已被废弃
      virtual void Post(const Location& posted_from ,
            MessageHandler* phandler ,
            uint32_t id = 0,
            MessageData* pdata = nullptr ,
            bool time_sensitive = false) {
         // Post是异步调用，因此只需将任务入队
         Message msg;
         msg.posted_from = posted_from;
         msg.phandler = phandler;
         msg.message_id = id;
         msg.pdata = pdata;
         messages_.push_back(msg);
      }

      // 消息处理循环
      bool ProcessMessage(int cmsLoop) {
         while(true){
            Message msg;
            // 从队列中获取一个待处理任务
            if(!Get(&msg, cmsNext))
               return !IsQuitting();
            // 分发处理
            Dispatch(&msg);
         }
      }

      // 分发处理
      void Dispatch(Message *pmsg) {
         pmsg->phandler->OnMessage(pmsg);
      }

      // PostTask是对Post的封装，此处省略task类型
      // 总之task中包含了数据，也包含了处理函数
      void PostTask(T task) {
         Post(RTC_FROM_HERE,
               &queue_task_handler_,
               /*id=*/0,
               new P(std::move(task)));
      }

      // PostTask使用举例
      // 将匿名函数转换为QueuedTask
      // PostTask(webrtc::ToQueuedTask([](){ /*...*/ });
   }
   
   // Post参数中，实现MessageHandler需要重写OnMessage函数，例如
   class BaseChannel : public rtc::MessageHandler {
      // 自定义处理过程
      void OnMessage(rtc::Message* pmsg) override;
   }

   // PostTask中使用的queue_task_handler_的类型，这里同样略去类型T
   class QueueTaskHandler : public rtc::MessageHandler {
      void OnMessage(rtc::Message* pmsg) override {
         auto* data = static_cast<T>(msg->pdata);
         // 还原成任务
         std::unique_ptr<T> task = std::move(data->data());
         if(!task->Run()){
            // ...
         }
      }
   }
   ```

   和前述方法不同，Send/Invoke方法， 虽然进行了线程切换，但是执行结果是同步返回的。代码引用如下。
   ```cpp
   class Thread {
      // 参数和Post没有本质区别
      virtual void Send(const Location& posted_from ,
            MessageHandler* phandler ,
            uint32_t id = 0,
            MessageData* pdata = nullptr) {
         // 第一步也是创建Message对象，略去
         Message msg;
         // ...

         // 第二部确保发送任务的线程，和Thread对象进行绑定。
         // 创建一个新的线程，可能用于代表当前线程完成计算任务
         // 该线程并不一定真正执行工作，具体要看ThreadManager、Thread::Current()
         AutoThread thread;
         // 获取当前可供执行任务的目标线程（此current指的是ThreadManager提供的可用线程）
         Thread* current_thread = Thread::Current();

         // 内部仍然是调用PostTask
         // 此处的Task有两个匿名函数
         PostTask(webrtc::ToQueuedTask(
            // 一个是处理任务
            [msg]()mutable {
               msg.phandler->OnMessage(&msg);
            },
            // 一个是完成后清理现场，并通知处理完成
            // 这样设计的目的，就是将两个匿名函数的功能分开，后者明显和业务无关
            [this,&ready, current_thread] {
               ready = true;
               current_thread->socketserver()->WakeUp();
            }
         ));

         // 开始等待同步返回
         while(!ready){
            current_thread->socketserver()->Wait();
         }
      }
   }

   // 其中AutoThread类型构造形如
   AutoThread::AutoThread(): Thread(SocketServer::CreateDefault(),false) {
      // 如果当前的线程未绑定线程对象
      if(!ThreadManager::Instance()->CurrentThread()) {
         // 初始化
         DoInit();
         // 将当前线程绑定到线程对象this
         ThreadManager::Instance()->SetCurrentThread(this);
      }
   }
   ```

2. 线程对外接口：为了将WebRTC内核代码和应用层进行隔离，WebRTC在对外提供接口时也使用了线程切换。而且提供接口的线程也只有信令线程和工作线程。例如PeerConnection接口，用于创建对端连接，媒体流。WebRTC将信令线程和工作线程的接口封装出了接口代理层。在使用接口代理层时，不必担心所调用的接口是哪个线程。对接口代理层的实现使用了宏。
   ```cpp
   // 部分宏相关代码示例
   BEGIN_SIGNALING_PROXY_MAP(PeerConnectionFactory)

   // 示例，实际宏参数更为复杂
   PROXY_METHOD4(
      refptr<PeerConnectionInterface>,
      CreatePeerConnection,
      const PeerConnectionInterface::RTCConfiguration&,
      std::unique_ptr<cricket::PortAllocator>,
      PeerConnectionObserver *
   )

   END_PROXY_MAP()


   // 利用这些宏，可以很方便的组装一个接口代理，预编译后会形成如下类型
   class PeerConnectionFactoryProxyWithInterface;
   typedef PeerConnectionFactoryProxyWithInterface
            PeerConnectionFactoryProxy;
   class PeerConnectionFactoryProxyWithInterface : 
            PeerConnectionFactoryInterface {
      // 忽略参数
      CreatePeerConnection(/*...*/) {
         MethodCall<C,r,/*...*/> call(c_, &C::method,/*...*/);
         return call.Marshal(RTC_FROM_HERE, signaling_thread_);
      }
   }

   // 其中MethodCall是对Handler和Message的实现
   class MethodCall : public rtc::Message, public rtc::MessageHandler {
      // 包装函数
      R Marshal(const rtc::Location& posted_from, rtc::Thread *t){
         Intenal::SynchronousMethodCall(this).Invoke(posted_from, t);
         return r_.moved_result();
      }

      void OnMessage(rtc::Message*) {
         Invoke(/*...*/);
      }

      void Invoke(/*...*/) {
         // r_是ReturnType
         r_.Invoke(c_, m_, /*...*/);
      }
   }

   // SynchronousMethodCall是Data、Handler的实现
   class SynchronousMethodCall : public rtc::MessageData, public rtc::MessageHandler {
      void Invoke(const rtc::Location& posted_from, rtc::Thread* t) {
         // 线程切换
         t->Post(posted_from, this, 0);
         e->Wait();
      }

      void OnMessage(rtc::Message*) {
         proxy->OnMessage(nullptr);
         e->Set();
      }
   }

   class ReturnType {
      void Invoke(C* c, M m, Args&& ...args){
         // 对PeerConnection来说，c->*m才是真正调用到PeerConnectionFactory::CreatePeerConnection()
         r_ = (c->*m)(std::forward<Args>(args)...);
      }
   }
   ```

1. 编解码模块：这部分的设计目标就是统一性，切换编解码器，对于上层业务应当是透明的。

### 部分类型设计
1. RtpPacket：封装对RTP协议的读写

## 实战内容
一个支持RTMP接入的服务器端、以及前端Demo

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
3. [SRS服务器](https://ossrs.net/lts/zh-cn/)：一个开箱即用、开源的视频解决方案，支持多种协议的多媒体数据。

3. SDP内容中常见的协议栈：UDP/TLS/RTP/SAVPF
   1. 协议最底层是UDP
   2. 在UDP之上，用DTLS进行传输层加密
   3. 再之上运行RTP，以及RTCP
   4. 最上层使用SAVPF进行反馈控制
4. sink：原意是沉没。在计算机行业中，对于流式数据场景，sink指代数据流的接收端。

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
3. [WebRTC中文网](https://webrtc.org.cn/)
4. [声网开发者社区](https://www.rtcdeveloper.cn/cn/community/)