---
title: "UE5学习：Cpp篇"
date: 2022-12-09T19:15:18+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
Unreal Engine另一个强大之处就在于它使用C++作为开发语言，和蓝图进行配合，能够发挥更大的威力。本文记录一些C++开发知识。
<!--more-->

## 基本流程
1. 构造函数：进行资源获取，基本的参数变量的设置
1. BeginPlay：对动画等和运行时相关的逻辑进行设定和启动的函数
1. Tick：每一帧都会进行调用的函数
1. SetPlayerInputComponent：将控制器的某些事件绑定当当前类型的处理函数

## 类型系统概述
1. Unreal Engine大幅拓展了C++的能力，并形成了自己的类型系统，添加C++类型时，需要遵照相应的框架规则。
1. 宏构成了Unreal Engine对C++扩展的一大部分，从类型生成到类成员的生成，都有宏的参与


## 常用宏
1. UE中使用的修饰作用的宏，一般的语法都类似于：
```c
// 修饰在前，key&value在后
MACRO([specifier, specifier, ...], [meta(key = value, key = value, ...)])
```
1. 一些重要的宏如下表
| 名称 | 使用位置 | 意义 |参数 |
| ----- | --- | --- | --- |
| UPROPERTY | 对类成员进行属性设置 |  | (UP::XXXenum，Category="在编辑器-细节面板中的名字") |
| UCLASS | 对类进行属性设置 | 用于创建被声明类的UClass | Transient、Blueprintable、BlueprintType等 |
| USTRUCT | 对结构体进行属性设置 | 用于创建被声明的类的UStruct | Blueprintable等
| UFUNCTION | 回调函数声明 | 回调类型的函数必须添加 | |

## 一些核心基类
1. ACharacter：角色类型通用的基类
| 成员名称 | 成员类型 | 含义 | 
| --- | --- | --- |
| GetCapsuleComponent() | 继承函数&emsp; | 获取碰撞胶囊组件，一般会进一步Init胶囊体大小 |
| GetMesh() | 继承函数 |获取网格体，用于进一步设置SkeletalMesh等 |
| bUseControllerRotationXXXX | 继承变量 | 指示变量：是否使用控制器的输入控制旋转角色 |
| SetupPlayerInputComponent() | 重写函数 | 用于将输入事件绑定到用户输入组件 |
| AutoPossessPlayer | 继承变量 | 用于记录该角色相机视角是否为初始视角（Player0） |
| AddMovementInput() | 继承函数 | 用于提供橘色的移动方向和移动量 |
| GetCharacterMovenent() | 继承函数 | 获取角色当前的运动组件 | 
| Jump() | 继承函数 | 设置角色进行一次跳跃（只是对速度、高度计算，动画需要用户控制 |
| GetActorLocation() | 继承函数 | 获取角色根组件的世界坐标 |

1. AController：角色控制通用的基类
| 成员名称 | 成员类型 | 含义 | 
| --- | --- | --- |
| InputComponent | 继承变量 | 默认为空的输入组件 |
| GetControlRotation() | 继承函数 | 获取当前控制器的旋转向量（欧拉角） | 
| GetCharacter() | 内联函数 | 获取当前控制器控制的角色 |

1. AGameModeBase：游戏模式基类
| 成员名称 | 成员类型 | 含义 |
| --- | --- | --- |
| PlayerControllerClass | 继承变量 | 默认玩家控制器 |
| DefaultPawnClass | 继承变量 | 默认角色 |

1. UAnimInstance：动画实例，多用于和蓝图配合编写更好的动画效果
| 成员名称 | 成员类型 | 含义 |
| --- | --- | --- |
| NativeInitializeAnimation | 继承函数 | 初始化动画 |
| NativeUpdateAnimation | 继承函数 | 每帧动画更新 |

1. UActorComponent：自定义组件类型最常用的基类之一，常用于实现一些具体游戏性逻辑，并被挂载到具体Actor上
| 成员名称 | 成员类型 | 含义 |
| --- | --- | --- |
| TickComponent | 继承函数 | Tick函数，每帧都会被调用 |

1. AActor：任何可放置、可Spawn的类型的基类
| 成员名称 | 成员类型 | 含义 | 注意 |
| AddActorWorldTransform() | 继承函数 | 用于对Actor做全局变换 | |
| AddActorLocalRotation() | 继承函数 | 用于对Actor做局部坐标系旋转 | |
| SetMaterial() | 继承函数 | 用于设置材质、材质实例 | |
| SetCollisionEnabled() | 继承函数 | 用于设置碰撞计算方式 | |
| AttachToComponent | 函数 | 将当前Actor设置连接到指定Component | 常用于将某物品绑定到人物、其他物品身上 |

1. UWorld：世界类型
| 成员名称 | 成员类型 | 含义 |
| SpawnActor() | 泛型函数 | 创建一个Actor |
| DestroyActor() |  | 删除一个Actor |

## 广泛继承的函数
| 名称 | 继承来源 | 含义 |
| --- | --- | --- |
| XXX::StaticClass() | | 创建并返回所属类型的静态实例 |
| GetWorld() | | 获取世界指针，常用于Spawn |

## 其他常用基本类型
| 名称 | 含义 | 注意 | 常用成员 |
| --- | --- | --- | --- |
| UStaticMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 静态网格体组件，即一个UStaticMesh的实例 | 可能被垃圾回收，需要用UPROPERTY进行标记 | SetupAttachment、SetStaticMesh、SetMobility |
| USkeletalMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 可动画化的网格体组件 | 也需用UPROPERTY标记 | SetAnimationMode、PlayAnimation、SetAnimInstanceClass、IsPlaying |
| UStaticMesh、USkeletalMesh | 静态、骨骼网格体底层存储类型 | 需设置给特定的component才能使用 | |
| FVector、FRotator | 向量、旋转子 | 常在SetXXXRotation/Location中使用 | |
| FString | 可变字符串 | 每个FString独立保存字符数组 | \*操作符 |
| USpringArmComponent | 摇臂类，常用于辅助第三人称相机 | 一般绑定到角色 | bUsePawControlRotation |
| UCameraComponent | 相机类型 | 常绑定到摇臂 | bUsePawnControlRotation |
| UInputComponent | 输入组件，将输入事件绑定到具体函数 | | BindAxis、BindAction | 
| UCharacterMovementComponent | 角色运动组件 | 在角色类中用Get函数获取 | 各种用于控制角色移动的参数，IsFalling()等运动状态函数 |
| UAnimSequence | 动画序列 | 实际还是用动画蓝图更方便 | |
| UBoxComponent | 盒组件 | 常用于进行简单的碰撞检测 | InitBoxExtent、OnComponent(Begin/End)Overlap.AddDynamic设置碰撞检测回调 |
| UParticleSystemComponent | 粒子系统组件 | 常用于烟雾、火焰等 | SetTemplate |
| UPointLightComponent | 点光源组件 | | SetLightColor、SetIntensity、SetSourceXXXX |

## 常用工具类/枚举类/函数
| 名称 | 含义 | 注意 |
| ------ | --- | --- |
| ConstructorHelper::FObjectFinder()&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 加载某种Object资源 | 构造函数中需要给出目标名称，加载的是不带Component后缀的原始资源 |
| LoadObject<>() | 加载某种object资源 | 构造函数中给出父类Outer（可null）、目标引用名或文件地址 |
| TEXT() | 对原始代码中的文本做正确处理 | 用于静态文件路径、文字打印等多种场景，本质是添加字符串前缀L |
| CreateDefaultSuboject<>() | 创建一个组件或者suboject | 构造函数中需要给出目标名称 |
| FRotationMatrix() | 创建旋转矩阵 | 构造函数中给出旋转角向量 |
| FTransform | 变换类型，包含所有变换方式 | |
| RotationMatrix::GetUnitAxis() | 获取旋转矩阵对单位轴向量旋转后的结果 | 参数是EAxis::X/Y/Z |
| Cast<>() | UE5类型体系下的转型 | |
| UE_LOG() | 日志宏 | 需要提供日志类型enum，日志级别enum |
| FStatic | 常量大全 | 避免在代码中使用魔数 |
| EInputEvent | 输入事件类型枚举类 | 在BindAction中使用，比如指定按键按下IE_Pressed还是松开IE_Released |
| EAutoReceiveInput::Type | 指示会被传递给当前角色、组件的是哪个玩家 | |
| EAnimationMode | 动画模式 | |
| ECollisionEnabled | 碰撞检测 | QueryOnly有一些bug，并不Only |
| EComponentMobility | 组件可移动性 | 静态、部分动态、全动态 |
| FMath | 数学工具类 | |
| FName | 公开名称，全局可见 | 常用于对骨骼网格体的某一部分进行检索、绑定 |
| FAttachmentTransformRules | 连接时的变换规则枚举类 | 用于连接时，指定连接运算方式 |

> 注1：类型系统中，所有非Class的内容，都是一种UObject。万物皆是UObject。

## 算法&数据结构
| 名称 | 类型 | 含义 | 注意 |
| --- | --- | --- | --- |
| TArray | 泛型容器 | 动态数组 | |
| TMap | 泛型容器 | 字典 | |
| TSet | 泛型容器 | 集合 | |

## 开发要点
1. 由于C++代码带来的变化不能像蓝图一样，自动显示在编辑器中，因此需要使用**Live Coding**功能，在不重启编辑器的情况下，编译并应用C++的罪行修改。快捷键是Ctrl+Alt+F11。
1. 常见的在项目设置中进行配置的内容：
    - 输入的事件绑定，按键事件、轴事件
    - 游戏模式
1. X轴正方向是默认的物体朝向、人脸朝向方向。因此在需要使用到面朝方向的计算时，也请使用GetUnitAxis(EAxis::X)。
1. DebugGame Editor模式并不能使用断点，需要使用Development Editor模式才可以。但是实话说断点并不好用，还是打日志吧。


## 常见功能示例
1. 添加自定义C++类：    
    1. 步骤：菜单栏→工具→新建C++类→选择合适的父类并创建
1. 

## 一些无奈
1. UE类型体系内的用户自定义类型，必须以XXXX.generated.h为最后一个头文件。


## 坑
1. C++类的更新无法在UE5编辑器内正常看到：动态编译的问题，推荐在偏好设置中，开启加载时强制编译
1. 打开已有项目，C++类在UE5编辑器内找不到了：动态编译的问题，推荐在偏好设置中，开启加载时强制编译
1. Windows下，Live Coding、UE5编辑器自定义类注释乱码：设置Windows语言为Unicode.UTF-8
1. UE5，在VS2019下开发时，IntelliSense性能差：安装引擎自带的UnrealVS插件（在引擎安装路径下，如D:/Epic Games/UE_5.0/Engine/Extras/UnrealVS/VS2019/UnrealVS.vsix）
1. 无法自动导入依赖所需要包含的头文件：
    - **尚未解决？**
1. UE体系内C++类型修改后，Live Coding后，运行仍然未更新：删除原对象，重新拖动对象到关卡内。
    - **是否有自动的办法？**
1. 继承函数一般都需要在函数刚开始时调用父类函数，例如
    ```cpp
    // 自定义动画类，动画更新
    void UMyAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
    {
        Super::NativeUpdateAnimation(DeltaSeconds);
    }
    ```
1. SetupAttachment用于构造函数中，AttachToComponent用于运行时修改。而且如果使用KeepRelativeTransform，要注意是否需要重置相对位置、旋转。
1. 想要删除一个C++类：需要先从VS中删除文件，再关闭UE编辑器，再删除Binaries文件夹，最后在VS中生成项目。**真的离谱**。
    - 如果不这么做，很有可能出现VS中莫名其妙的错误，
1. 据说UE5目前的烹专该内测，使用QueryOnly时，仍然不能保证是只检测而不会禁止移动，因此需要具体设置静态网格体的算法，设置如下
    ```cpp
    if (type == ECollisionEnabled::QueryOnly) {
        // 无视阻挡
		staticMesh->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);
	}
	else {
        // 阻挡
		staticMesh->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Block);
	}
    ```
1. 回调函数必须添加宏描述UFUNCTION()，例如碰撞检测回调
    ```cpp
    UFUNCTION()
	void OnOverlapBegin(class UPrimitiveComponent* OverlappedComp
		, class AActor* OtherActor, class UPrimitiveComponent* OtherComp
        , int32 OtherBody, bool bFromSweep, const FHitResult& hit);

	UFUNCTION()
	void OnOverlapEnd(class UPrimitiveComponent* OverlappedComp
		, class AActor* OtherActor, class UPrimitiveComponent* OtherComp
        , int32 OtherBody);
    ```
1. GetBounds函数，在不同类型中名字可能不同，但是注意获取到的origin和boxIntent都是局部坐标系的值。并且都是包含当前节点及其子节点的所有组件在内的整体包围盒。使用前确认在哪一级组件上。
1. Overlap检测，注意对应的Actor在创建时，关于碰撞盒和网格体的缩放的控制：**尚未很好的解决**
    - BoxComponent的InitBoxIntent：只控制碰撞盒的大小（默认32，和坐标不是同一种单位），但同时也受BoxComponent的缩放参数的控制
    - StaticMeshComponent的SetRelativeScale3D：作为BoxComponent的子组件时，只控制网格体的缩放
    - 灵活使用UE编辑器，可以先在C++中编码，然后在场景中拖出来一个，之后在编辑器的细节窗口中调整参数，直到调整好之后，再将参数带回C++代码中。
1. 如果一个物体Spawn时就和另一个物体重叠，无法触发Overlap重叠事件：**尚未解决**

## 参考
1. [【虚幻5】【不适合小白观看】用C++来进行基于UE5的游戏开发（含动画蓝图）](https://www.bilibili.com/video/BV17Q4y1Y7fr)
1. [官网：C++编程 虚幻引擎编程开发的相关信息](https://docs.unrealengine.com/5.0/zh-CN/programming-with-cplusplus-in-unreal-engine/)
1. [UE5-c++教程 01~05](https://www.bilibili.com/video/BV1be41137Kp)
1. [知乎专栏：UE从点Play开始](https://zhuanlan.zhihu.com/p/512249255)
1. [Unreal Engine C++ Advanced Dark Souls Boss Fight System](https://www.youtube.com/watch?v=ANzEGECpd0g)
1. [UE5 C++ Tutorial | Introduction to Unreal Engine 5 with C++ in less than 90 Minutes](https://www.youtube.com/watch?v=nvruYLgjKkk&list=PL-m4pn2uJvXHL5rxdudkhqrSRM5gN43YN)
1. [UE4静态/动态加载资源方式](https://zhuanlan.zhihu.com/p/266859719)
1. [【教程】虚幻5教程 斯坦福专用课程 UE4 & C++ 专业游戏开发教程 24.5小时 中文字幕](https://www.bilibili.com/video/BV1nU4y1X7iQ)
1. [UE4 UCLASS宏和可用宏参数](https://zhuanlan.zhihu.com/p/148098617)