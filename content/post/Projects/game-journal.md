---
title: "游戏开发：开发日志"
date: 2023-04-18T22:17:24+08:00
categories:
- 游戏开发
tags:
- 系列开坑
- 游戏开发
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/game.jpg
draft: true
---
本文是各类游戏学习的开发方向日志，写给自己看的记录内容，内容包括UE5源代码学习，各类游戏设计学习，个人项目的思路回顾
<!--more-->
# DemoZero开发日记
> 《我的第一款生存类缝合怪游戏》
## 综述
### 主要玩法：
&emsp;&emsp;在随机生成的地图中和好友一同探索世界，采集资源，建立并发展生存点，招募NPC伙伴，发展科技，对抗敌对生物。
### 背景设定
&emsp;&emsp;别太中二
### 要素：
1. 支持联机
    - 服主自动获得管理员身份、其他人按管理员分配权限继承顺位
    - 兼顾单人和多人难度，难度主要受生存点规模、人数（人类、电脑NPC）影响
1. 角色成长
    - 基于加点：智力&精准度、力量&耐力、敏捷&适应力，作为初期游戏职业
    - 支持洗点（大记忆清洗术、大记忆恢复术）
1. 开放沙盒，随机生成地图：
    - 地形随机生成，高山、湖泊、洞穴，不同地形有不同的优势劣势
        - 柏林噪声算法
        - 提供一定程度上修改地形的能力
            - 地形第一次随机生成之后，就将会完全存储到文件，此后每次只加载，不生成
            - 地形修改主要有削平和隆起两种
    - 建筑随机生成：一些完好的房屋和破旧遗址
    - 资源随机生成：一次性生成和定期刷的资源
1. 季节和昼夜系统
    - 虽然有四季，但是夏季气温很低，只有在部分地形有适合耕种的环境，其他地方只能架设温室
1. 生存建造：
    - 提供预设建筑和自定义建筑
    - 建筑、工作台提供不同的能力
1. 招募NPC伙伴：
    - 通过特定建筑（电台、灯塔）等，招募NPC伙伴，也可以表决流放NPC
    - NPC可以指派任务
    - 可以建造机器人
1. 战斗和敌对生物：
    - 单兵和多人武器、生存点防守武器
    - 根据地形、季节和昼夜等条件限制刷怪
    - 怪物掉落
### 开发注意
1. 优先完成游戏性，最后填充美术资源
1. 考虑mod支持
    - [知乎：小团队的游戏作品如何添加MOD支持？](https://zhuanlan.zhihu.com/p/451445398?utm_id=0)
### 法律
1. 注意所有工具链是否可商用
    1. 操作系统？
    1. Visual Studio需要购买Professional订阅，在年盈利超过100w美元后需要升级为Enterprise订阅
    1. UE5可以在盈利后再支付，100w美元之后，所有超过1w美元收入的月份，缴纳收入的5%
    1. 模型：Blender
    1. 平面素材？
    1. 音乐？
### 参考
1. Banished
1. 凯撒大帝
1. kenshi
1. minecraft
1. 饥荒
1. 这是我的战争

## 第三节点
1. 目标：继续第二周的目标
    1. 新目标：
        - [*] 从5.1更新到5.2
        - [ ] 尝试使用5.2中提供的过程内容生成
        - [ ] 完成对地图的动态生成，保存，加载功能
        - [ ] 重构地形生成器代码，提供更好的设计模式
        - [ ] 重新引入puerts，尝试将部分逻辑在脚本中进行编写
            - [ ] 完成C++、蓝图、TS互操作
    1. 关键点：
        1. 5.1到5.2
            1. 不要在代码里写和版本绑定的include，比如include "unreal engine 5.1/...."
            1. 5.1中使用的puerts作为插件，无法直接支持5.2，需要等待社区进行升级
                - 重新评估脚本系统？
            1. Landscape中的CollisionComponent的获取方式有变化
            1. 原位转换其他内容正常
        1. Procedural Content Generation：
            - 该部分更倾向于替代Foliage（植被工具），提供更好的泛用性，并具备实时使用内容，而不是地形生成
            - 参考:[Unreal Engine 5.2 Procedural Content Generation Tutorial](https://www.youtube.com/watch?v=_Mk3x5W8tHY)、[Procedural Content Generation in UE5 | GDC 2023](https://www.youtube.com/watch?v=aoCGLW53fZg)
    1. 支线：
        - 为项目添加提高蓝图可用性的插件：[蓝图 Visual Studio Integration Tool](https://www.unrealengine.com/marketplace/en-US/product/362651520df94e4fa65492dbcba44ae2)
1. 时间安排
    1. 6月
> 以周为单位的计划安排不够合理

## 第二周
1. 目标：实现基础的角色和联机
    1. 细化目标：
        - [x] 角色能够加入Dedicated Server，并且不同角色~~~仅加载~~~各自的区域，服务器需要加载所有区域并进行计算
        - [x] 修改地形系统，使得能够根据角色进行区域加载
        - [ ] 角色拥有物品系统，能够收集、丢弃资源
            - 暂时用text3dactor代替物品，也需要为其添加碰撞（仿mc）
        - [ ] 尝试Linux、Windows双版本
    1. 关键点：
        - Enhanced Input：
            - IM和IA配合使用，IA绑定到具体的控制（相当于信号和槽），IM负责给IA映射按键
            - 想要使用组合按键，要在IM中进行配置，设置组合，组合方式和所有相关键的触发方式有关，可以根据需要调整（都按下、有顺序按等等）
        - UMG:
            - 注意先创建UUserWidget子类，然后再单独创建蓝图，并选择继承刚刚的C++父类（而不是从C++父类直接派生蓝图子类）
        - 关于蓝图：
            - 不排斥蓝图，但是也不要过于依赖，对于一些能提高效率的场景，多使用蓝图是完全可以的。比如对于动画，角色运动、输入动作的基本配置，使用蓝图更方便
        - 参考Minecraft，每秒20个Tick，将渲染帧和游戏逻辑帧分离。
            - Tick暂时用定时器确定
        - GameMode和GameState：
            - 将一些游戏基础的逻辑移动到GameMode和GameState中进行存储和控制
        - 联机注意：
            - GameMode是不会向客户端复制的，位于listenServer上的玩家，仍然可以调用GameMode实例中的方法和属性，但是其它纯client的玩家，但凡使用GameMode实例中的方法，均会抛出异常（因为这个类并不会向其他客户端进行扩散）
                - 注意在编辑器的启动模式中区分单机（无服务器），监听服务器（ListenServer），客户端（启动纯Client并自动启动一个Dedicated Server）
            - 需要使用GameState进行同步
                - 控制地形的同步生成
                - 存档同步（为了简化：一律采用ds + client模式，只不过ds只有房主才能启动）
                    - 本地存档保存所有只属于玩家的信息（名称，id，位置等），不保存任何公共信息（地图、建筑）
                    - 登陆后由服务器负责下发地图
                    - 优化空间？
                - login控制（AGameModeBase::Login）
            - 最重要的几条思路：
                - 分清楚代码要在哪里执行：并使用HasAuthority（对本地和远程玩家进行区分）、IsRunningDedicatedServer（对是否位于服务器进行区分，收编辑器状态影响）、IsRunningClientOnly（对是否位于客户端进行区分）
                - 注意判断方式和编辑器模式有关，这也算是一个坑
                - 其他判断方式
                    - GetGameInstance()->IsDedicatedServerInstance()
            - 新的Player加入时，GameState等可能尚未同步完成，此时在BeginPlay中使用相关内容可能造成错误的初始化。
        - 语言注意：
            - 转型问题：注意float和uint16不能直接转型，uintXX是虚幻5定义类型
            - 不要使用FVector2i（怀疑删除，即使引用了对应的GeometryCore也无法引入），使用FIntVector/2D/3D。
            - TArray、TMap、TSet不是线程安全的
        - 考虑到工具链成熟情况，选择暂时使用PuerTS，以ts作为脚本语言
    1. 疑问：
        - 即使是Dedicated Server，也需要完整的创建Landscape，否则移动角色会出现问题（调入虚空）
    1. 核心知识
        - UE5最新的增强输入
        - 参考
            - [UE5 -- EnhancedInput(增强输入系统)](https://zhuanlan.zhihu.com/p/470949422)
            - [【UE5】角色动画蓝图，制作八方向混合空间](https://www.bilibili.com/video/BV1TG4y1V75k/)
            - [Creating Custom Landscape Importers](https://docs.unrealengine.com/5.1/zh-CN/creating-custom-landscape-importers-in-unreal-engine/)
            - [UE5 保存和加载游戏](https://docs.unrealengine.com/5.1/zh-CN/saving-and-loading-your-game-in-unreal-engine/)
            - [Unreal Engine 5.1 - Enhanced Input - Multi-Key Actions](https://www.youtube.com/watch?v=KguFlvQHWH8)
            - [Set Up HUD & UI Widget - Unreal Engine 5 Tutorial [UE5]](https://www.youtube.com/watch?v=YCQ1heoaILY&t=8s)
            - [UE5中GameMode、玩家角色等的设置与获取](https://zhuanlan.zhihu.com/p/560971474)
            - [UE4原子操作与无锁编程](https://www.cnblogs.com/kekec/p/13750892.html)
            - [在编辑器中运行多玩家选项](https://docs.unrealengine.com/5.1/zh-CN/play-in-editor-multiplayer-options-in-unreal-engine/)
            - [UE5中的网络同步能力和同构服务器框架（上）](https://zhuanlan.zhihu.com/p/621339344)
            - [《图解UE4渲染体系》Part 0 引擎基础](https://zhuanlan.zhihu.com/p/493147444)
            - [强力推荐Alex Forsythe系列视频： Multiplayer in Unreal Engine: How to Understand Network Replication ](https://www.youtube.com/watch?v=JOJP0CvpB8w)
            - [UE4学习之路：Puerts安装和调试](https://zhuanlan.zhihu.com/p/456267138)
            - [Puerts & ReactUMG 环境搭建（二）](https://zhuanlan.zhihu.com/p/397369095)
## 第一周
1. 目标：实现随机地形生成，注意地形生成应由服务器端控制
    1. 细化目标：
        - [x] 动态生成landscape并能卸载
        - [x] 地形根据柏林噪声生成
    1. 心态：
        - 一定要保持好心态，UE5的所有代码都在本地，有不理解的事情，尝试先google，google不到可靠的结果，就自己去看代码，也不是坏事
        - 思路：把地形工具，放到运行时使用，入口是LandscapeSubsystem，主要是FLandscapeConfigHelper，多看一下LandscapeEditor的相关流程代码。
            - 怀疑有游戏额PostEdit、PostImport函数，里面执行了一些必需的逻辑
        - 代码中做了很多优化的事情，可以适当跳过
    1. 关键点：
        - 虽然ALandscape是标准的地形类型，但是该类型和地形编辑器高度绑定，使用起来有一定困难可以参考相关博客、以及ALandscape的实现，自行实现一个地形类型
        - 编译时链接失败有很大概率是未在build.cs中添加对应模块（ALandscape需要添加Landscape模块），否则反射机制可能未对相应类型起作用。所在模块可以从官网对类的介绍中看到（Module、Header、Source）
        - ALandscapeProxy不能被直接Spawn，它是抽象类
        - GEngine->GetWorld()不能乱用，实际上，对于World，在Standalone、Editor、Dedicated Server模式下都各有不同
        - 创建ALandscape之后仍然需要调用函数，完成自然的接缝
        - 类型
            - APartitionActor
                - ALandscapeProxy
                    - ALandscape
                    - ALandscapeStreamingProxy
        > 参考：《Unreal Engine 4 Scripting with C++ Cookbook.pdf》400~404页
    2. 一些可以复用的内容：
        - 获取HeightData最大值和中间值：LandscapeDataAccess::MidValue MaxValue
        - 给Actor起唯一的名字：FActorLabelUtilities::SetActorLabelUnique
    3. 坑：
        - 官方的FMath::PerlinNoise，不能在整数点采样，否则结果必为0
        - 填充高度图TArray\<uint16\>时，需要按先y后x的遍历顺序（否则高度图和import时的读取方式不匹配会无法接缝）
        - 由于设计者故意将Landscape设计为静态内容，所以任何运行时的SetActorLocationRotation都需要先对所在ALandscapeStreamingProxy.RootComponent.Mobility进行修改
    4. 尚未解决：
        - 接缝？
        - 物理碰撞网格不对？
        - 构造、BeginPlay，到底哪些操作能在这两个函数中进行，为什么GetWorld有时崩溃，有时没事
        - Accessor、UTexture2D、TArray、数据处理、使用方式
        - 虽然可以参考LandscapeEditor，但是有很多在Private中的头文件，是无法使用的，强行include也会报错，这导致了HeightCache、XYOffsetCache无法使用。
        - 第一个地块和其他地块连不上，目前的办法是，构建之后再把第一个删掉，重新创建（首次创建和后续流程确实有区别，盲猜应该是哪里设置了RelativeLocation）
        - ALandscapeProxy.Import会构建法线，但是针对LandsdcapeConfigHelper中的FindOrCreateNew并不会
        - 随处可见HeightmapScaleBias，这个Bias是干嘛的
    5. 核心数学：
        ```c++
        // 高度图中RGBA通道的作用
        HeightmapInfo.HeightmapTextureMipData[0][HeightTexDataIdx].R = HeightValue >> 8;
        HeightmapInfo.HeightmapTextureMipData[0][HeightTexDataIdx].G = HeightValue & 255;
        HeightmapInfo.HeightmapTextureMipData[0][HeightTexDataIdx].B = FMath::RoundToInt(127.5f * (Normal.X + 1.0f));
        HeightmapInfo.HeightmapTextureMipData[0][HeightTexDataIdx].A = FMath::RoundToInt(127.5f * (Normal.Y + 1.0f));
        ```
    6. 核心知识：
        - UE5的游戏启动、运行、结束流程，以及流程中所有参与类型的生命周期
        - 参考：
            - [UE4 Runtime Landscape](https://www.cnblogs.com/LynnVon/p/11776482.html)
            - [UE5引擎 PC端的Landscape渲染浅分析](https://blog.csdn.net/qq_29523119/article/details/125532143)
            - [4.27官方流程图](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Framework/GameFlow/)
            - [Landscape generation #1 - mesh generation | Unreal Engine C++](https://www.youtube.com/watch?v=sNZ2g4qah28)
            - [Youtube: 用程序生成地图（E01：导言）](https://www.youtube.com/watch?v=wbpMiKiSKm8&list=PLFt_AvWsXl0eBW2EiBtl_sxmDtSgZBxB3)
            - [Creating a Lanscape using C++ or the Python API](https://forums.unrealengine.com/t/creating-a-lanscape-using-c-or-the-python-api/504997)
            - [Landscape Creation C++ UE4.27](https://stackoverflow.com/questions/71424982/landscape-creation-c-ue4-27)
            - [UE5 C++笔记，使用柏林噪声生成像素地形](https://www.bilibili.com/video/BV153411G7CB)
            - [地形系统初探](https://cloud.tencent.com/developer/article/2071226)



## 总体参考
1. [知乎专栏：深入UE5剖析源码，浅出游戏引擎架构理念](https://zhuanlan.zhihu.com/insideue4?utm_id=0)
1. [知乎专栏：少亿把趁手的兵器-UE5拓展工具箱](https://www.zhihu.com/column/c_1309300932140589056)
1. [源码看《饥荒》-知乎](https://www.zhihu.com/column/c_1298055007057526784)
## 大牛关注
1. YouTube：
    - Sebastian Lague：系列视频Coding Adventure