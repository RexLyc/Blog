---
title: "UE5学习：设计和术语篇"
date: 2022-12-27T22:02:50+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
本章节主要记录一些游戏设计，以及一些游戏设计相关术语。
<!--more-->

## 视角切换
1. 第一人称：一般的，此时相机绑定到角色（骨骼模型），设置合适的相对旋转和相对位移。相机、角色都会跟着鼠标的移动而旋转。
1. 第三人称：一般的，此时相机绑定到一个摇臂，而摇臂进一步绑定到角色整体上（而非骨骼模型），设置合适的相对旋转和相对位移。角色不跟随鼠标移动旋转，相机会在摇臂上绕角色旋转。

## 术语
1. 旋转：
    - Pitch：绕x轴旋转，即俯仰角的改变
    - Yaw：绕y轴旋转，即左右方向，水平角度的改变
    - Roll：绕z轴旋转，滚转
    > 注1：一般的，对于旋转，以相机坐标系为参考，是以x轴为屏幕向右，y轴屏幕向上，z轴从屏幕指向外侧，具体看左右手坐标系
    > 注2：对于位移，则是以全局坐标系为参考，此时默认x是模型前方，y是右方，x&y构成地平面，z则指向天空