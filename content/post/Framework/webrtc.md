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
### 信令服务器
&emsp;&emsp; 信令服务器提供通信双方在业务层上的管理、以及信息的交换。完成用户创建房间、进入/退出房间，交换双方IP和端口等功能。

## 实战内容
1. 

## 其他内容
### 常见问题和方法
1. 减少数据量：压缩、SVC技术、Simulcast技术、动态码率、甩帧
2. 适当增加时延：缓冲队列
3. 提高网络质量：降低丢包、延迟、抖动。NACK/RTX、FEC前向纠错、JitterBufer、NetEQ、拥塞控制
4. 快速评估带宽：Goog-REMB、Goog-TCC、NADA、SCReAM

## 参考
1. 《WebRTC音视频实时互动技术》
2. [P2P技术原理浅析](https://keenjin.github.io/2021/04/p2p/)