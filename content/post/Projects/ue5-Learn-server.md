---
title: "UE5学习：服务器开发篇"
date: 2023-02-02T20:18:49+08:00
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
联机功能能给大多数游戏增加非常多的乐趣，因此有必要在一开始就考虑联机功能，考虑联机服务器的开发。
<!--more-->

## 服务器概述
1. 网络模式
    1. 独立运行：独立运行的本机游戏，不接受远程客户端连接
    1. 纯客户端：只作为客户端，必需连接到服务器才能使用多人游戏模式
    1. 监听服务器：拥有本地玩家，同时也作为服务器为其他人提供连接，相当于多人联机中的主持人角色
    1. 专属服务器：只运行服务器逻辑的部分，不具备本地玩家
1. 关键概念：
    1. GameMode：在专属服务器模式，GameMode只由服务器管理
    1. GameState Actor：在专属服务器模式下，从服务器向客户端同步一些GameMode属性时，使用GameState进行同步
    1. Actor复制：大部分Actor默认不启用复制，需要将Actor的bReplicates变量设置为true，开启Actor复制
        - Actor有不同的复制层级，比如角色类的ACharacter会默认开启运动复制bReplicateMovement
    1. 变量复制：使用UPROPERTY种的Replicated宏等方式，指定变量进行复制
    1. RPC：远程过程调用，有服务器、客户端、广播三种方式，需要制定可靠或不可靠属性（即保证到达和不保证到达）
1. 基本原理：
    - UE5引擎通过数据同步，维持不同类型的实例（或者一部分属性）在多台机器上具有相同的副本

## 编译和交叉编译
1. 为什么要编译：官方的二进制安装包只能进行本地开发，无法进行服务器开发
1. 为什么要交叉编译：考虑到服务器经常是Linux系统，而开发机器一般是Windows，所以官方也提供了交叉编译方案。
1. 流程
    1. 在官网查找交叉编译文档，下载并安装必需的软件（工具链），注意一定要**优先查阅英文文档**
    1. 在CMD中运行Setup.bat，这一步还会下载各种必需的依赖项，初次配置的话，大概在20GB左右
    1. 运行GenerateProjectFiles.bat，注意报错信息和平台支持情况
    1. 编译看硬件情况，ue5.1版本，使用5900x花费约1小时
1. 坑：
    1. 项目路径中不能有空格（也不建议有奇葩字符），否则必定报错
    1. 建议是在SSD中放置源码、编译引擎。使用HDD真的要命
    1. 使用自己编译的Unreal Editor之后，经常发生，明明只是构建自己的程序，却几乎把半个引擎rebuild了（2800个文件，接近完整的一半），这是个**已知大坑，尚未解决**，注意以下几点：
        1. 任何时候不要使用**清理、重新生成**，这会导致自行编译的虚幻引擎的库被删掉，如果需要完全重新生成自己的项目：删除Binaries、Build、DerivedDataCache和Intermediate文件夹，以及sln文件。用uproject重新生成sln
        1. 任何时候，打开sln后、先注意是否将自己的项目设置为启动项目，即使已经设置，也尽量再设置一下
        1. 只使用**生成**
    1. 二进制安装的UE5编辑器在崩溃时会有一个很好用的调用栈信息，可以很高效地用来推测一些严重Bug。但是自行编译的版本，在崩溃的时候只会弹出一个Fatal Error。**尚未解决**，只能多用调试模式了。

## 核心编程
### 属性复制
1. 流程
    1. 根据需要，为需要同步的字段设置UPROPERTY(replicated)
    1. 实现函数GetLifetimeReplicatedProps
    1. 正常使用
1. 示例代码
```cpp
// h
UCLASS()
class YourCharacter: public ACharacter {
public:
    // 待同步变量
    UPROPERTY(replicated,EditAnywhere,BlueprintReadOnly,Category=Input,meta=(AllowPrivateAccess="true"))
    float sizeBase;

    void GetLifetimeReplicatedProps(TArray< FLifetimeProperty >& OutLifetimeProps)  const;
}

// cpp
void YourCharacter::GetLifetimeReplicatedProps(TArray< FLifetimeProperty >& OutLifetimeProps) const {
    // 注意这里填的是所在类的类型
	DOREPLIFETIME(YourCharacter, sizeBase);
}
```

### RPCs
1. 流程
    1. 根据需要，选择Client、Server、NetMulticast模式，并设定Reliable、Unreliable
    1. 编写实现函数：XXXX_Implementation
    1. 在合适的地方进行调用
1. 示例代码
```cpp
// h
UCLASS()
class YourCharacter: public ACharacter {
private:
    UFUNCTION(Server, Unreliable)
    void ClientRPCHelloWorld();
}

// cpp
void AThirdZeroCharacter::BeginPlay()
{  
	Super::BeginPlay();
    // 调用是原始名称
	ClientRPCHelloWorld();
}

// 实现时添加_Implementation后缀
void AThirdZeroCharacter::ClientRPCHelloWorld_Implementation() {
	UE_LOG(LogTemp, Log, TEXT("another client come up online"));
}
```

## 实用枚举、函数、类
| 名称 | 类型 | 含义 | 注意 |
| --- | --- | --- | --- |
| GetNetMode() | 函数 | 获取当前的网络模式 | 用于判断代码运行在哪种环境下(客户端、服务器等) |
| SetReplicatingMovement() | 函数 | 设置是否同步角色移动 | bReplicateMovement是继承的private变量，只能用函数修改 |

## 打包项目
1. 方式
    1. 在C++工程中，添加XXXBuild.Target.cs，进行详细配置
    1. 在UE5编辑器中，点击：平台->Windows/Linux->选择打包的类型（Game/Client/Server等）
    1. 在UE5编辑器中，点击：平台->Windows/Linux->打包项目
    1. 选择要导出的内容、文件夹

## 运行服务器
1. 常用选项
    1. -log：添加一个输出日志的终端

## 参考
1. [Unreal官方Github代码仓库](https://github.com/EpicGames/UnrealEngine)
1. [官网：C++编程 虚幻引擎编程开发的相关信息](https://docs.unrealengine.com/5.0/zh-CN/programming-with-cplusplus-in-unreal-engine/)
1. [联网和多人游戏](https://docs.unrealengine.com/5.1/zh-CN/networking-and-multiplayer-in-unreal-engine/)
1. [官网：网络多人游戏基础-概述](https://docs.unrealengine.com/5.1/zh-CN/networking-overview-for-unreal-engine/)
1. [官网：设置专用服务器](https://docs.unrealengine.com/5.1/zh-CN/setting-up-dedicated-servers-in-unreal-engine/)
1. [Youtube: Building Unreal Engine 5 from Source Code](https://www.youtube.com/watch?v=wGq-XK8mffg)
1. [UE4随笔：意外的重新编译或生成引擎源码](https://zhuanlan.zhihu.com/p/393760492)
1. [解决UnrealEngine项目编译后每次都需要重新编译的坑 ](http://www.morecpp.cn/ue-opensource-project-trigger-compile/)
1. [Engine always rebuilding on the project build](https://forums.unrealengine.com/t/engine-always-rebuilding-on-the-project-build/294050/12)