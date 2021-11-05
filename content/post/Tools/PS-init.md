---
title: "Photoshop：基础篇"
date: 2021-11-05T20:43:52+08:00
categories:
- 实用工具
- 平面后期
tags:
- 实用工具
- 摄影
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/PS.jpg
math: true
---
Photoshop作为平面后期处理工具的集大成者，是提升照片观赏效果的绝佳搭档。本系列是纯新手的学习笔记，仅追求特定拍摄素材和场景的处理技巧。
<!--more-->
# 色彩
从红到紫，可见光范围内有无限多种色彩，其中最重要的三要素就是色调（色相）、饱和度、明度。人眼看到的任一彩色光都是这三个特性的综合效果，其中色调与光波的频率有直接关系，亮度和饱和度与光波的幅度有关。
# 色彩空间概述
1. sRGB
    - 目标是让显示器、打印机拥有通用颜色标准
2. Adobe RGB
    - 目标是让显示器显示更多颜色
3. CMYK：C(青)、M(品红)、Y(黄)、K(黑)
    - 彩色印刷时使用的颜色模型
> Adobe RGB $\supset$ sRGB, Adobe RGB $\supset$ CMYK
>
> sRGB-CMYK $\ne \varnothing$ 且 CMYK-sRGB $\ne \varnothing$。
# 色彩空间选择
1. 一般来说拍摄时无脑选Adobe RGB就好啦。
2. 在Photoshop中，有如下区域涉及到色彩空间
    - 视图 -> 校样设置
        1. CMYK：如果未来需要印刷，则应当选择工作中的CMYK（观感上主要是绿色将不在鲜艳）
            - 此时会出现色域警告，指示部分颜色可能无法正确印刷
        2. 显示器RGB & sRGB：如果仅在网络中使用，则使用这两种均可
    - 编辑 -> 颜色设置-> RGB模式
        1. Adobe RGB：用于需要打印的情况
        2. sRGB：用于在网络传输的情况
    - 编辑 -> 配置文件 & 转换成配置文件
        1. Adobe RGB：用于需要打印的情况
        2. sRGB：用于在网络传输的情况
3. 无脑的选择方式：
    - 网络：视图中选择显示器RGB，其他位置全部Adobe RGB
    - 打印：视图中使用CMYK，其他位置全部Adobe RGB
# 图片格式
1. JPEG：最常用的格式之一，**有损压缩**，颜色为8位/通道。
2. TIFF：无损压缩甚至不压缩，占用空间很大，可以保留RAW过来的16位色彩空间。
3. RAW：存储原始的底片信息，拥有16位的色彩空间。
# Camera Raw插件(12.3)
## 概述
- 这个插件是将RAW格式导入Photoshop的必经之路，注意需要在这里的存储选项中，选择好对应的转换格式和色彩空间。
![不同版本的Camera RawUI可能有所不同](/images/Photoshop/CameraRawSaveOption.jpg)
- 本身也有非常实用的调色工具、校准工具等，不需要很重的调色的话，在这里就能完成相关的调整。
## 颜色矫正
- 白平衡：以无色相（R=G=B）的颜色为参考，恢复照片原本颜色的过程。核心标准是50%的灰度。实际操作过程中，应当根据审美调整。
- 分离色调处理：
    - 普通的白平衡无法处理，明暗部亮度差距过大时可能的颜色误差：比如暗部偏蓝、亮部偏黄
    <center><img src = "/images/Photoshop/CameraRawSaveColorFix1.jpg"></center>

    - 此时需要将亮部暗部，本应是无色相的颜色尽量调整到无色相的状态，再进行白平衡。
- 目标调整工具：
    - 可以在图片中直观的调整特定颜色所代表的颜色段的特性
    <center><img src = "/images/Photoshop/CameraRawTargetAdjustTool.png"></center>
    
    - 除了曲线工具，**混色器等工具**中也存在目标调整工具
## 曝光
## 色阶
## 直方图
# 参考资料
- [摄影后期拍摄技巧与后期调色，PS人像风景摄影后期教程](https://www.bilibili.com/video/BV1gb4y167Sh?from=search&seid=10473265529199477631&spm_id_from=333.337.0.0)
- 