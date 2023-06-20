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
## 语言相关特性
### UObject
#### 反射

#### 序列化

#### GC
1. GC有两个入口
   1. UObject
   2. FGCObject
2. 最佳实践
   1. 成员应尽可能都添加```UPROPERTY()```，空的也行，以保证获得完整的UObject处理，以及运行时正确的GC表现

#### UHT和UBT
1. UnrealBuildTool
   - 作用：扫描所有的头文件，记录任何从上次编译后有修改的且具有至少一个反射需求的编译单元，将该编译单元加入UHT的工作列表中。
2. UnrealHeaderTool
   - 作用：在UBT确定了需要更新的编译单元后开始工作，为指定的头文件添加反射等所需的各类代码，每个编译单元最终生成一个.inl文件。
   - 注意：
        1. 由于UHT并没有具备完整的C++解析能力，因此除了```WITH_EDITOR``` / ```WITH_EDITORONLY_DATA```之外，应当尽量避免在UPROPERTY等宏附近使用条件编译```#ifdef```。
        2. 受UHT限制，无法在类定义内使用using、typedef，如果有需要，在类外使用。<font color=#ff6644>*待确认？*</font>
        3. 受UHT限制，不能对重载函数（overload）使用UFUNCTION，对重写函数（override）使用UFUNCTION时也要求宏参数保持一致。<font color=#ff6644>*待确认？*</font>

### 委托和代理
1. 概述：UE中委托和代理广泛存在，这两种模式在事件、渲染等机制中被广泛使用
2. 委托（delegate）步骤：
   1. 核心宏：用于描述委托函数元属性的```UDELEGATE()```、用于描述委托函数的函数签名的```DECLARE_XXX_DELEGATE_XXX```。
   2. 使用宏定义委托函数，如```MyActionDelegate```
   3. 在必要时进行绑定```MyActionDelegate.Bind()```，以及委托调用```MyActionDelegate.Execute()```
3. 代理（proxy）：
## Actor
1. 基本说明：所有能被放置到场景中的东西，基本都是AActor及其派生
2. Actor的生命周期：
   1. 创建步骤：
      1. 方式一：加载。从磁盘加载、从Editor复制
      2. 方式二：生成。```SpawnActor()```
   2. 正常使用
   3. 销毁：
      1. ```Destroy```：标记为等待销毁
      2. ```EndPlay```：保证销毁
   > 参考：[UE5.2 Actor 生命周期](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-actor-lifecycle/)

## Component
1. 基本说明：用于附加给Actor，以实现各种功能
2. Component的生命周期：
   1. 创建步骤：
      1. 方式一：构造函数中，```CreateDefaultSubObject```
      2. 方式二：其余位置动态生成，```NewObject```
   2. 创建后的一些重要步骤：
      1. 使用```NewObject```方式创建的Component，如果需要能够逐帧更新并影响场景，必须**调用注册**```RegisterComponent```，渲染和物理状态都会在注册中进行初始化。
   3. 正常使用
   4. 销毁：
      1. 仍想保留组件，但是不希望再参与更新，可以通过**取消注册**```UnregisterComponent```
         1. 

   > 参考：[UE5.2 组件](https://docs.unrealengine.com/5.2/en-US/components-in-unreal-engine/)

## Editor

## GamePlay Framework

## 帧

## 网络通信

## 其他子系统

## 其他参考
1. [知乎专栏：InsideUE系列](https://www.zhihu.com/column/insideue4)
2. [Unreal Property System Reflection](https://www.unrealengine.com/zh-CN/blog/unreal-property-system-reflection)