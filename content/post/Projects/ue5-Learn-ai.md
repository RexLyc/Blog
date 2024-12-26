---
title: "UE5学习：AI篇"
date: 2023-07-06T10:57:23+08:00
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
AI是游戏机制中构成玩法的非常重要的一部分，本篇记录UE开发中的AI编写方法。
<!--more-->
## 导航
1. Nav Mesh
   - 一类特殊的网格体，在网格体中的区域是寻路算法认定可以进入的区域
2. 蓝图
   - AI Move To（蓝图函数）：基于Nav Mesh的寻路
   - Move To Location Or Actor（蓝图函数）：不需Nav Mesh，也可以进行寻路
3. 实用工具
   1. 在编辑器内按P键可以预览导航网格
      ![导航网格预览](/images/ue/NavMesh.jpg)
4. 坑
   - 导航网格的生成策略有3种，静态，动态仅修改（不会生成新的），动态。其中静态生成，则只有在编辑器内预览的部分才有，程序不会在运行时进行任何更新。
        > 如果会在运行时生成地形、生成建筑，则需要酌情实用动态仅修改、动态这两种。
5. 参考：
   1. [UE5.2 Modifying the Navigation Mesh](https://docs.unrealengine.com/5.2/en-US/overview-of-how-to-modify-the-navigation-mesh-in-unreal-engine/)

## 行为树
1. 黑板：用于在代码蓝图中共享各类键值数据
2. 行为树设计蓝图：
    - 概述：行为树具有自己的设计编辑器。行为树的使用依赖于```ACharacter```、```AIController```，从对应角色蓝图、控制器蓝图中可以选择想要使用的行为树。
    - 主要有6种类型节点：
        1. 任务（Task）：紫色，代表可以执行的蓝图程序，内部返回```true```、```false```代表任务执行成功与否。结束时需要调用```FinishExecute()```。根据AI触发方式区分：
            1. 接收Tick触发（蓝图名称Receive Tick AI）
            2. 被行为树调用触发（Receive Execute AI）
            3. 被行为树终止（Receive Abort AI）
        2. 装饰器（Decorator）：实际上是一个条件判断节点，用于决定是否执行该节点后续内容。引擎提供了一些常见的装饰器，也可以自定义装饰器（用蓝图编写自定义的测试条件）。常见装饰器节点：
            1. 黑板键检测：对在黑板中配置的键进行判断
            2. 视锥检测：对某两个物体在游戏运行时，三维空间内的可视性进行检测
            3. 路径检测
            4. 冷却检测：俗称cd时间
        3. 服务（Service）：当该节点运行Tick等事件时，执行一系列蓝图程序。这里的Tick频率可以调整，是帧率无关的。服务内的内容，不需要调用```FinishExecute()```。主要的服务触发事件有
            1. Tick触发
            2. AI激活事件
            3. AI取消激活事件
        4. 选择器（Selector）：具有若干个子节点，从左至右依次执行，直到有一个子节点返回**成功**
        5. 顺序（Sequence）：具有若干个子节点，从左至右一次执行，知道有一个子节点返回**失败**
        6. 并行（Parallel）：同时执行所有子节点
        > 注1：任务、装饰器、服务是基础节点。后三个是组合节点。
        
        > 注2：Selector、Sequence节点，如果其子节点是Task，且内部蓝图代码没有调用FinishExecute，则无法成功执行相应逻辑。

        > 注3：至少在UE5.2版本，装饰器、服务的定义方式，是在任意Task、Composite节点，右键，选择添加装饰器、添加服务。而创建则需要先选择所继承的基类。如下图
        
        ![UE5.2 装饰器、服务的定义](/images/ue/BTNewDecoratorEtc.jpg)

        ![UE5.2 装饰器、服务的使用](/images/ue/BTDecoratorEtc.jpg)
        
3. 行为树设计示例：
   - 一个近战策略行为：定期搜索可攻击目标、锁定仇恨、移动到目标位置、尝试攻击、等待攻击冷却
## 参考
1. [UE5.2 Artificial Intelligence](https://docs.unrealengine.com/5.2/en-US/artificial-intelligence-in-unreal-engine/)