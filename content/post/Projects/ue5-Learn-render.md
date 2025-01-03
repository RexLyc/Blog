---
title: "UE5学习：渲染和视觉效果篇"
date: 2023-06-14T22:01:55+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
draft: true
---
画面是UE的强项，本篇记录所有UE相关的视觉效果内容。
<!--more-->
## 材质和纹理
1. 区分纹理和材质：
    - 纹理（Texture）：一般来说就是一个图片。
    - 材质（Material）：一系列资源和可视化处理的组合，可能是颜色纹理、法向、光照方程等等。
2. 自定义材质：[参见蓝图章节]({{< relref "/content/post/Projects/ue5-Learn-Blueprint.md#材质" >}})
    - 常见效果的制作原理：
        1. 水面：材质坐标映射受时间影响，水面上下做多种材质，[推荐参考：UE_材质_水材质详解](https://www.bilibili.com/video/BV1od4y137CD)
        2. 落叶：粒子系统 + 材质坐标映射受事件影响
        3. 菲涅尔效应（Fresnel Effect）：物体表面反射率和视线方向相关
3. 材质模块化
    1. 自定义材质节点：在材质编辑器画板内右键创建，提供自定义的输入输出，可以编写自定义HLSL渲染表达式
        ![自定义节点](/images/ue/ShaderCustomNode.jpg)
    2. 材质函数：
        - 相比于材质表达式，材质函数的复用性更强
    3. 结合材质实例，并使用输入参数（标量、向量）：提升对材质的控制能力
        - 使用各类参数，可以让同一种材质，具有不同的材质实例，展示不同的材质效果
            ![从材质创建材质实例](/images/ue/MaterialInstance.jpg)
4. 一些内置节点
    1. 微光（Glimmer/Glittering）：通过周期性调整亮度等，达到微光闪烁的效果
    2. 菲涅尔反射（Fresnel Effect）：反射衰减指数、基础反射率
    3. 柏林噪声
5. 地形材质
    - 概述：地形材质相对复杂，往往由多种材质混合而成，主要的混合参数有
        - 材质权重
        - 材质alpha通道
        - 地形高度
## 着色器

## 实用功能
1. 预览节点：开发着色器过程中，对指定节点预览输出，可以查看每一个节点的输出变化

## 参考
1. [博客园专栏：剖析虚幻渲染体系](https://www.cnblogs.com/timlly/p/13512787.html)
2. [2020年 秋 犹他大学 计算机图形学课程（中英字幕）](https://www.bilibili.com/video/BV11f4y1c78w/)