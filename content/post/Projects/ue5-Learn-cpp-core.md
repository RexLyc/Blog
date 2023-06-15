---
title: "UE5学习：Cpp引擎核心篇"
date: 2023-06-12T15:53:45+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
math: true
---
本篇将会对重要的引擎基础架构、运行时、编辑器等各方面的流程、原理进行总结。
<!--more-->
## UObject
### 反射

### 序列化

### GC
1. 最佳实践
   1. 成员应尽可能都添加```UPROPERTY()```，空的也行，以保证获得完整的UObject处理，以及运行时正确的GC表现

### UHT和UBT
1. UnrealBuildTool
   - 作用：扫描所有的头文件，记录任何从上次编译后有修改的且具有至少一个反射需求的编译单元，将该编译单元加入UHT的工作列表中。
2. UnrealHeaderTool
   - 作用：在UBT确定了需要更新的编译单元后开始工作，为指定的头文件添加反射等所需的各类代码，每个编译单元最终生成一个.inl文件。
   - 注意：
        1. 由于UHT并没有具备完整的C++解析能力，因此除了```WITH_EDITOR``` / ```WITH_EDITORONLY_DATA```之外，应当尽量避免在UPROPERTY等宏附近使用条件编译```#ifdef```。


## Editor

## 启动

## 帧

## 网络通信

## 其他子系统

## 其他参考
1. [知乎专栏：InsideUE系列](https://www.zhihu.com/column/insideue4)
2. [Unreal Property System Reflection](https://www.unrealengine.com/zh-CN/blog/unreal-property-system-reflection)