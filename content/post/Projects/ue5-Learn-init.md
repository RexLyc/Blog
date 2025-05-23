---
title: "UE5学习：开坑篇"
date: 2022-04-21T23:16:22+08:00
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
本篇是独立游戏制作系列的游戏引擎开坑篇。由于虚幻引擎开源，且语言以C++为主，因此选择虚幻5引擎作为学习起点。
<!--more-->
## 学习内容
1. 三维关卡示例
1. 代码运行原理
1. 测试和发布
1. 实用技巧

## 基础术语
1. [界面基本构成](https://docs.unrealengine.com/5.0/zh-CN/unreal-editor-interface/)
    - 默认界面可能变化，根据需要自行调整
    - 主工具栏可以选择编辑模式、内容快捷创建、播放模式控制、场景构建等重要功能
    - 关卡视口的左上角可以选择关卡视口的观察模式（透视、正交）、右上角选择对模型的修改方式
    - 引擎已内置对版本管理的支持，在文件->源码管理（Source Control）
    - 大纲（Outlier）面板（一般于右上角），展示&编辑关卡中的所有内容
1. [一些基本术语](https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-terminology/)
    - 蓝图：完善的Gameplay脚本系统，主要是基于可视化的开发流程
    - 对象：虚幻引擎中最基本的类，C++中UObject是所有类的基类
    - Actor：可以放到关卡中的任何object、例如相机、模型、玩家出生点，C++中AActor是所有Actor的基类
        - Pawn：特指由人类或AI控制的Actor，也就是常说的游戏角色或者NPC
        - Character：特指人类控制的Pawn
    - 类：用于实现特定的Actor或对象的行为，C++、蓝图项目中均可以创建类
    - 投射（Cast）：尝试把某个动作施加给特定的Actor，可能成功也可能失败。例如控制角色在特定位置回血
    - 组件（Component）：将某个功能添加给Actor时使用
    - 玩家控制器：获取玩家输入，并将其转化到对游戏的控制，C++类型PlayerController
    - AI控制器：控制NPC行为，C++类型AIController
    - 玩家状态：保存游戏参与者在游戏中的状态，C++类型PlayerState
    - 游戏模式：设置游戏运行规则，C++类型GameMode
    - 游戏状态：保存游戏中需要维护的信息，多人游戏中，该信息同样需要被同步到所有玩家的电脑，C++类型GameState
    - 其他：
        - 笔刷Brush：描述3D形状的Actor
        - 体积Volumes：带有边界的3D空间，
        - 关卡Level：定义的gameplay区域，包含玩家可以看到并与其交互的所有内容。关卡保存为.umap文件，也被称为地图Map
        - 世界World：构成游戏的所有关卡的容器，处理关卡流送和Actor的动态生成
1. 开发术语：
    - PIE：在编辑器中运行
    - SIE：在编辑器中模拟
## 基本操作
1. 视图：
    - 鼠标左键、中键、右键均可以控制不同方式的视图变换
    - 在按下鼠标任何一个案件的同时，可以按QEWASD等键进行移动，此时滑动滚轮可以调整移动速度
    - Q、W、E、R：分别是选中、平移、旋转、缩放模式
    - 在关卡视口中，灵活使用左上角的：透视、光照、模型，选项，来使开发更方便
    - 有些视口会隐藏一些选项卡，可以在各个视口的设置中寻找一下（比如右侧大纲默认隐藏了世界场景设置World Settings）
    - 一些特殊但有用的视图：
        - 光照、线框、可视性碰撞、玩家碰撞、优化视图模式中的若干视图
1. 操作模式：
    - 选择模式
    - 地形编辑模式
1. Actor：
    - Ctrl+D：复制一个Actor
    - Alt+拖拽：复制一个Actor
    - Shitf+拖拽：移动Actor并锁定视角
    - End：将选中物体底部对齐到下方Actor（如地板）
1. 在不同项目之间导入导出资源文件
    - 在想要导出的项目的Content（或指定folder处）
    - 右键选择迁移（Migrate），选择导出文件夹，一般直接导出到想要导入的项目即可
    - 资源将被自动加载进去
1. 易用性：
    - 书签（Bookmark）：在任意位置使用Ctrl+数字组合键，设置一个书签点，之后按数字键即可回到该位置。

## 平衡蓝图和C++
1. 用蓝图快速开发原型
2. 用C++优化其中的性能要求较高的部分

## 参考资料
1. [官方文档（中文）](https://docs.unrealengine.com/5.0/zh-CN/)
2. [官方Learning Library](https://dev.epicgames.com/community/learning)
3. [油管Unreal Engine 5 Beginner Tutorial - UE5 Starter Course!](https://www.youtube.com/watch?v=gQmiqmxJMtA)
4. [Unreal Engine 5 Beginner Tutorial - UE5 Starter Course 2022](https://www.youtube.com/watch?v=k-zMkzmduqI)
5. [知乎博主系列文章：丛越](https://www.zhihu.com/people/li-xiang-da-sheng/posts)
6. 图形学相关：
   1. [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/V1X7411F744)
   2. [GAMES202-高质量实时渲染](https://www.bilibili.com/video/BV1YK4y1T7yY/)
7. [Unreal Engine C++编程文档](https://italink.github.io/ModernGraphicsEngineGuide/04-UnrealEngine/0.%E5%9F%BA%E7%A1%80%E7%BC%96%E7%A8%8B/)
8. 官方推荐数字资产
   1. [【UE5】Hillside虚幻5.2官方炫技的重磅资产，63G大小诚意十足](https://www.bilibili.com/video/BV1tm4y1t7hu/)