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

图中展示了一对一通信的整体架构和流程。其中
1. 检测客户端上可用的音视频设备
2. 开始采集
3. 录制
4. 打开同信令服务器的通信
5. 创建```RTCPeerConnection```对象，绑定音视频数据
6. 获得当前客户端在NAT映射中外网端口/IP（通过STUN/TURN进行NAT穿透）
7. 通过信令服务器将Call/Called（主叫被叫）地址发送给对端
8. 通过RCPeerConnection建立连接

### 信令服务器
信令服务器提供通信双方在业务层上的管理、以及信息的交换。完成用户创建房间、进入/退出房间，交换双方IP和端口等功能。所谓信令，也就是包含这些管理事件的信息。信令服务器需要对信令进行顺序限制，这种限制称为时序关系。

信令也分为客户端发起和服务端发起两个种类，一个简单的信令设计如下
1. 客户端发起：join、leave、message，代表客户端加入、离开、发送消息（用于同步各种元信息）
2. 服务端发起：joined、left、full，代表确认客户端加入、确认离开、房间满员

信令服务器的基本实现就是这样了。实际上，只要你能有办法达成这些接口，无论你使用什么网络协议都是可用的。通常情况下，还是会使用基于TCP等可靠连接的上层协议，如HTTP/HTTPS、WS/WSS。

### 

## 实战内容
一个支持RTSP接入的服务器端、以及前端Demo

## 其他内容
### 术语
1. STUN：Session Traversal Utilities for NAT，NAT会话穿越实用工具
2. TURN：Traversal Using Relay NAT，中继穿越NAT

### 常见问题和方法
1. 减少数据量：压缩、SVC技术、Simulcast技术、动态码率、甩帧
2. 适当增加时延：缓冲队列
3. 提高网络质量：降低丢包、延迟、抖动。NACK/RTX、FEC前向纠错、JitterBufer、NetEQ、拥塞控制
4. 快速评估带宽：Goog-REMB、Goog-TCC、NADA、SCReAM


## 参考
1. 《WebRTC音视频实时互动技术》
2. [P2P技术原理浅析](https://keenjin.github.io/2021/04/p2p/)