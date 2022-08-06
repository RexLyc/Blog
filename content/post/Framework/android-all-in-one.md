---
title: "Android开发合集"
date: 2022-07-28T17:01:23+08:00
categories:
- 计算机科学与技术
- 安卓
tags:
- 持续施工
- 安卓
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/android.jpeg
# draft: true
---
项目中偶尔需要一些Android开发，在此记录一下
<!--more-->
## 概述
Android是基于Linux开发的一款优秀的操作系统，尤其适用于在手机端使用，目前也是用户数量最多的操作系统。本文从实用角度出发，给出一些使用的项目代码记录。
## Android Studio安装
1. 最简单粗暴的办法，在HTTP Proxy设置中，直接使用Auto-detect proxy settings，然后用本机的任何上网代理即可。这也是**最稳定**的办法。
    - 但一定要在**第一次安装运行前就准备好代理**，否则可能出现包安装不完整导致的一系列非常奇怪的问题。
    - android studio的代理（SDK）和gradle代理（各类第三方库）并不是同一回事，在settings.gradle中还可以选择所使用的代码仓库的URL
1. 或者你也可以选择配置镜像：由于Android SDK存在版权问题，这里要搜索下可以使用的镜像源（清华TUNA在撰写本文时并不能使用）。可以使用中科院开源协会，填写如下图。
![Android Studio Proxy设置](/images/postImage/AndroidStudioProxy.jpg)
1. 由于Android Studio更新较快，新版本很容易出现问题，建议在出现难以解决的问题的时候，可以使用

## 编译篇
### SDK
### SDK Build Tool
### Gradle配置
1. 概述
    1. 全局配置是gradle.properties，位于用户目录的.gradle/下
    1. 项目配置是gradle-wrapper.properties，位于项目下
1. 代理：注意http和https都**必须写上**
    ```python
    #代理服务器IP/域名
    systemProp.http.proxyHost=127.0.0.1
    #代理服务器端口
    systemProp.http.proxyPort=10809
    #代理服务器IP/域名
    systemProp.https.proxyHost=127.0.0.1
    #代理服务器端口
    systemProp.https.proxyPort=10809
    ```
## 代码应用
### 权限获取

### 跨线程通信
- 关键词：
    - Handler：核心方法如sendMessage、handleMessage，用于发送、处理一个消息
    - Looper：执行Handler的handleMessage所在的事件循环线程
    - Message、Bundle：发送消息的数据结构
- 基本流程
    - \[可选\]自定义Handler，尤其是handleMessage函数
    - 创建Handler、创建Looper，启动Looper
    - 将Handler发给不同的线程
    - 各线程内自己组Message并发送
- 示例代码
    - 待补充
### Camera2和TextureView
- 关键词
    - 待补充
    - 补充一张Camera2的类图
- 基本流程
    - 为TextureView添加SurfaceViewListener
    - 由于TextureView的SurfaceView在Listener的onAvailable阶段才可用，所以初始化需要放到该位置
    - 根据需要调用capture拍照、或者setRepeatingRequest预览
- 难点
    - 记得控制画面长宽
    - 相机权限获取
- 参考
    - [CAMERA2 API 采集视频并SURFACEVIEW、TEXTUREVIEW 预览](https://www.freesion.com/article/3644114052/)
    - [Camera2 Camera1](https://yeungeek.github.io/2020/01/24/AndroidCamera-Orientation/)
    - [预览同时ImageReader获取图片](http://codingsky.com/doc/2022/4/19/842.html)
    - [基于Camera2实现边录制视频边实时分析图片](https://blog.csdn.net/m0_37697747/article/details/122077631)
    - [Android Camera2 全屏预览+实时获取预览帧进行图像处理](https://blog.csdn.net/qq_38743313/article/details/101557079?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-3-101557079-blog-122077631.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-3-101557079-blog-122077631.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=6)：这一篇讲了一些坑和处理方法
    - [ImageReader 丢帧卡顿](https://blog.csdn.net/xuhui_7810/article/details/104402300)
## 坑
1. 新建项目提示类似于：plugin com.android.application not found in any repositories
    - 尚不清楚原理，但是大概率和gradle有关，建议删除默认gradle目录(C:/Users/你的名字/.gradle/)，完全重新下载。
    ![删除gradle目录](/images/postImage/delete_gradle_to_redownload.jpg)
    - 其他的可能性包括：仓库中确实没有该版本、gradle和插件版本不兼容、gradle和gradle所用java版本不兼容（Gradle7.0+要求java11）、网路代理错误等
    - [参考](https://metapx.org/plugin-with-id-com-android-application-not-found/)

## 参考资料
- [2022最新Android基础视频教程](https://www.bilibili.com/video/BV19U4y1R7zV)
- [Google官方文档](https://developer.android.google.cn/reference)
    - Android Studio已经将文档融合进Android SDK源代码中，只需要安装项目所使用版本的SDK时同时安装其源代码即可