---
title: "游戏开发：资产篇"
date: 2023-07-30T20:07:00+08:00
categories:
- 游戏开发
tags:
- 系列开坑
- 游戏开发
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/game.jpg
draft: true
---
本文章记录游戏开发中所有的数字资产的制作方式。尽量做到便宜好用。
<!--more-->

## 美术
1. 2D
2. 3D
   1. Blender（3.6.1）：
      1. 常用操作
         | 操作名称 | 快捷键 | 注意 |
         | --- | --- | --- |
         | 移动旋转缩放 | G、R、S | 输入后再按X、Y、Z可以固定轴 |
         | 复制 | Shift + D | |
         | 旋转视口 | 鼠标中键 | 按住Alt可以吸附旋转到一些固定角度！ |
         | 移动视口 | Shift + 鼠标中键 |  |
         | LoopCut（增加环） | Ctrl + R | |
         | PropotionalEdit（衰减编辑） | 先按O，再进行其他变换 | 将一个变换施加给附近元素，类似于PS的液化 |
         | 切换所选元素级别 | 1、2、3 | 分别是点、线、面 |
         | 切换模式 | Tab | 编辑模式、物体模式 |
         | 切换模式（轮盘） | Ctrl + Tab | 会呼出一个轮盘，选择所有的六种模式 |
         | 面挤出 | E | |
         | 添加元素 | Shift + A |  |
         | Apply应用（变换） | 选择元素后Ctrl + A | 选择需要的变换（如Visual Geometry To Mesh，应用所有Modifier） |
         | 合并元素（物体） | Ctrl + J | 将右侧Collection中的元素合并 |
         | 分离元素（点线面） | 选中后按P | 可根据选择、材质等分离 |
      1. 雕刻模式下操作
         | 操作名称 | 快捷键 | 注意 |
         | 调整衰减编辑区域大小 | 按住F并左右移动鼠标调整 | |
         | 调整衰减羽化 | 按住Shift + F并左右调整鼠标 |
         | Clay粘土 | C | 使模型隆起（圆形笔） |
         | Clay Strip粘土条 |  | 使模型隆起（方形），使用时如果再按住Ctrl/Shift，则是模型凹陷 |
         | Smooth平滑 | Shift + S | 使模型平滑 |
         | Grab抓取 | G | 拖拽模型 |
      1. Modifier说明
         | Modifier名称 | 作用 | 注意 |
         | Mirror | 创建镜像元素 | 可以按照轴，也可以按照一个物体对象镜像 |
         | Subdivision Surface | 细分表面 | 原有物体的点成为控制点，可以进一步通过LoopCut增加控制点 |
         | Remesh | 重新网格化 | 根据给定算法和参数重新计算物体网格，**注意计算量** |
      2. 参考：
         - [【教程】Blender + UE5 游戏角色建模材质绑定动画全流程](https://www.bilibili.com/video/BV1MY4y1X7gn/)
         - [【中文字幕】UE5与Blender完整游戏环境制作工作流程视频教程](https://www.bilibili.com/video/BV1Ft4y1T7KW)
         - [Blender学习笔记：一个Q版人物](https://space.bilibili.com/27462787/channel/collectiondetail?sid=902549)
## 音乐
1. SuperCollider
   1. 参考：
      - [Github: awesome SuperCollider](https://github.com/madskjeldgaard/awesome-supercollider)
2. Sonic Pi
   1. Windows10：安装后无法启动

## 资源网站
1. [ANATOMY360：一个提供精细三维重建模型的网站](http://anatomy360.info/) 