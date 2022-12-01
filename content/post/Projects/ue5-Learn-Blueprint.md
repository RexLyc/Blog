---
title: "UE5学习：蓝图篇"
date: 2022-12-01T22:47:06+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
本文是学习使用蓝图方式建立一个游戏工程的教学视频的课程笔记。
<!--more-->
## 概述
&emsp;&emsp; 蓝图是UE5面向非程序员类型的开发者推出的GamePlay脚本系统，对于一些简单目标，蓝图的功能已经足够完成。
## 全局后处理
1. 类型：PostProcessorVolume
1. 可调整内容：曝光、镜头、颜色分级、渲染功能等等
1. 影响范围：只影响相机、而且只影响相机位于Volume体积内时（除非设置了无限范围）

## 材质
1. 基本用法：拖动一个材质到静态网格上
1. 创建自己的材质：
    - 在Material文件夹内新建材质，双击进入编辑
    - 编辑节点，连接，取消连接（Alt并单击）
    - 点击“应用”，进行材质编译
1. 材质节点：非常复杂，由很多的种类，这里记录一些基础的
    - Constant3Vector：RGB颜色常用节点，输出到基础颜色
    - TextureSampler：纹理采样器，可以输出RGB到粗糙度、高光度、各向异性、自发光等等
    - LinearInterpolate：线性插值，用一种值去混合两种值，如和TextureSampler搭配制作锈迹效果
        - 线性插值的输入数据A和B必须有同样的维度


## 参考
1. [Unreal Engine 5 Beginner Tutorial - UE5 Starter Course 2022](https://www.youtube.com/watch?v=k-zMkzmduqI)