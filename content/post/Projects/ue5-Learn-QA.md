---
title: "UE5学习：问题与解决方案篇"
date: 2023-01-10T20:56:53+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
本文介绍一些UE5实际开发中常见问题和解决方案
<!--more-->
## 图形图像篇
1. 模型重叠时的闪烁问题：
    - 当两个面共面时，会出现闪面现象。使用UE4材质中 Pixel Depth Offset 节点，进行像素偏移，达到共面不闪面的效果。
    - **待解决**
    

## 参考
1. [UE4/5 开发面试题：UE & 图形学 - Wychao09的文章 - 知乎](https://zhuanlan.zhihu.com/p/579078025)