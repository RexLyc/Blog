---
title: "UE5学习：Cpp概述篇"
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
1. AActor体系
   1. 构造函数：进行资源获取，基本的参数变量的设置
   2. BeginPlay：对动画等和运行时相关的逻辑进行设定和启动的函数
   3. Tick：每一帧都会进行调用的函数
   4. SetPlayerInputComponent：将控制器的某些事件绑定当当前类型的处理函数
2. UWorld

## 类型系统概述
1. 关于C++
   1. Unreal Engine大幅拓展了C++的能力，并形成了自己的类型系统，添加C++类型时，需要遵照相应的框架规则。
   2. 宏构成了Unreal Engine对C++扩展的一大部分，从类型生成到类成员的生成，都有宏的参与
2. UObject
   ```cpp
    // 一个典型的UObject定义
    #pragma once
    #include 'Object.h'
    #include 'MyObject.generated.h'
    UCLASS()
    class MYPROJECT_API UMyObject : public UObject
    {
        GENERATED_BODY()
    };
   ```
   - 通过UCLASS宏标记一个类需要经过UObject的一系列相关处理
   - 任何UObject子类，均不应使用new&delete，而应当使用NewObject&CreateDefaultSubClass生成，使用ConditionalBeginDestroy释放（不建议手动调用）
   - GENERATED_BODY宏会用于生成一系列引擎必需代码，主要是各种必需的成员函数
   - 每个UObject都有自己的类默认对象（Class Default Object），主要用于记录各个成员应当被赋予的默认值
   - 自动支持：成员自动初始化、序列化、网络通信、垃圾回收等能力
   - MYPROJECT_API用于将该类进行导出，等价于dllexport
   - UE的垃圾回收机制和UE的智能指针不能同时工作
3. UClass
   - UObject系统处理过程中，需要为每个类型添加反射的相关代码，尤其是需要保存各个成员的名称、类型信息，这些信息被封装到UClass内
   - 从UObject、UField、UStruct一路继承过来
> 参考：
> - [官方文档：UE5.2 UObject](https://docs.unrealengine.com/5.2/zh-CN/objects-in-unreal-engine/)
> - [腾讯游戏学堂：UE4 引擎基础类说明](https://gwb.tencent.com/community/detail/119428)
> - [知乎专栏：UE4 UObject 对象概念](https://zhuanlan.zhihu.com/p/419769230)
> - [《InsideUE4》UObject（三）类型系统设定和结构](https://zhuanlan.zhihu.com/p/24790386)
> - [官方文档：UE5.2 UField](https://docs.unrealengine.com/5.2/en-US/API/Runtime/CoreUObject/UObject/UField/)

## 常用宏
1. UE中使用的修饰作用的宏，一般的语法都类似于：
    ```c
    // 修饰在前，key&value在后
    MACRO([specifier, specifier, ...], [meta(key = value, key = value, ...)])
    ```
1. 一些重要的宏如下表
    | 名称 | 使用位置 | 意义 |参数 |
    | ----- | --- | --- | --- |
    | UCLASS | 对类进行属性设置 | 用于创建被声明类的UClass | 
    | UPROPERTY | 对类成员进行属性设置 |  | BlueprintReadOnly、replicated等 |Transient、Blueprintable、BlueprintType等 |
    | USTRUCT | 对结构体进行属性设置 | 用于创建被声明的类的UStruct | Blueprintable等 |
    | UFUNCTION | 回调函数声明 | 回调类型的函数必须添加，使其拥有反射能力 | Client等 |
    | UENUM | 枚举声明 | 修饰enum class | BlueprintType |
    | TEXT | 任何需要使用多字节字符串的位置 | 避免乱码 | 参数就是你想要使用的字符串 |
    | DECLARE_DELEGATE_XXX | UCLASS之前 | 自定义委托 | 参数是委托函数的签名情况，还可以进一步用UDELEGATE修饰 |
    | DECLARE_MULTICAST_DELEGATE_XXXX | UCLASS之前 | 自定义事件 | 为指定类型提供广播事件机制 |
    | GET_MEMBER_NAME_CHECKED | 语句内 | 用于编译期检查某个类是否存在某个成员属性 | 类名、成员名 |
1. 常见宏参数含义
    | 名称 | 所属宏 |  含义 |
    | --- | --- | --- |
    | Blueprintable | UCLASS | 允许在编辑器中从该类型派生蓝图 |
    | BlueprintType | UCLASS、UENUM | 允许该类型在蓝图中作为类型使用（创建变量） |
    | BlueprintCallable | UFUNCTION | 允许函数在蓝图中调用 |
    | EditAnywhere | UPROPERTY | 在属性窗口，为原型（蓝图）和实例设置。 |
    | EditDefaultsOnly | UPROPERTY | 在属性窗口，仅为原型设置 |
    | EditInstanceOnly | UPROPERTY | 在属性窗口，仅为实例设置 |
    | VisibleAnywhere | UPROPERTY | 在原型、实例的属性窗口中均可见 |
    | VisibleDefaultsOnly | UPROPERTY | 属性仅可在原型设置中可见 |
    | VisibleInstanceOnly | UPROPERTY | 属性仅可在实例设置中可见 |
    | BlueprintReadOnly | UPROPERTY | 属性仅可在蓝图中读取（GetXXX） |
    | BlueprintReadWrite | UPROPERTY | 属性可以在蓝图中读写 |
    | Category | UPROPERTY | 该属性在属性面板中的分类名称 |
    | AllowPrivateAccess | UXXX(meta=(~="?")) | 使private成员支持从蓝图操作（默认不允许） |
    | MetaClass | UXXX(meta=(~="?")) | 限制出现在编辑器的下拉列表中的C++子类型名称 |
    | BlueprintImplementableEvent | UFUNCTION | 函数可以在蓝图中实现 |
    | BlueprintNativeEvent | UFUNCTION | 表明该函数会有默认C++实现，但同时允许蓝图重写 |
    | BlueprintAssignable | UPROPERTY | 多播委托专用，可以在蓝图中添加委托绑定 |
> 注：
> 1. EditXXX限定符自带VisibleXXX效果，二者不能同时使用

> 参考：
> - [官方文档：UE5.2 uproperty 描述符列表](https://docs.unrealengine.com/5.2/en-US/unreal-engine-uproperty-specifiers/)
> - [官方文档：UE5.2 uclass 描述符列表](https://docs.unrealengine.com/5.2/en-US/class-specifiers/)
> - [官方文档：UE5.2 Metadata usable in UPROPERTY](https://docs.unrealengine.com/5.2/en-US/API/Runtime/CoreUObject/UObject/UM_3/)
> - [UE4 UPROPERTY Explained](https://www.youtube.com/watch?v=ZPJtFa9srXw)
> - [官方文档：UE5.2 Metadata Specifiers](https://docs.unrealengine.com/5.2/en-US/metadata-specifiers-in-unreal-engine/)
> - [官方问答那个： UE5.2 Delegates and Lambda Functions](https://docs.unrealengine.com/5.2/en-US/delegates-and-lamba-functions-in-unreal-engine/)

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

2. AController：角色控制通用的基类
    | 成员名称 | 成员类型 | 含义 | 
    | --- | --- | --- |
    | InputComponent | 继承变量 | 默认为空的输入组件 |
    | GetControlRotation() | 继承函数 | 获取当前控制器的旋转向量（欧拉角） | 
    | GetCharacter() | 内联函数 | 获取当前控制器控制的角色 |

3. AGameModeBase：游戏模式基类
    | 成员名称 | 成员类型 | 含义 |
    | --- | --- | --- |
    | PlayerControllerClass | 继承变量 | 默认玩家控制器 |
    | DefaultPawnClass | 继承变量 | 默认角色 |

4. UAnimInstance：动画实例，多用于和蓝图配合编写更好的动画效果
    | 成员名称 | 成员类型 | 含义 |
    | --- | --- | --- |
    | NativeInitializeAnimation | 继承函数 | 初始化动画 |
    | NativeUpdateAnimation | 继承函数 | 每帧动画更新 |

5. UActorComponent：自定义组件类型常用的基类之一，常用于实现一些具体游戏性逻辑，并被挂载到具体Actor上。继承自UActorComponent应当是不渲染的组件。
    | 成员名称 | 成员类型 | 含义 |
    | --- | --- | --- |
    | TickComponent | 继承函数 | Tick函数，每帧都会被调用 |

6. USceneComponent: 自定义组件类型常见的基类之一，常用作需要进行变换的节点的挂载节点。USceneComponent本身是**不渲染**的，但会保存平移、旋转、缩放能力

7. UStaticMeshComponent:静态网格体组件
    | 成员名称 | 成员类型 | 含义 |
    | --- | --- | --- |
    | bGenerateOverlapEvents | 成员变量 | 触发碰撞检测事件 |

8. UPrimitiveComponent： 较为复杂自定义组件基类。以它为基类的组件，大都是需要进行渲染的组件。常见的派生类有UMeshComponent、UShapeComponent
    | 成员名称 | 成员类型 | 含义 |
    | --- | --- | --- |
    | CreateSceneProxy | 继承函数 | 创建并返回FPrimitiveSceneProxy实例 |
    | GetDynamicMeshElements | 继承函数 | 用顶点、材质创建面元并传递给收集器 |
    | OnActorPositionChanged | 继承函数 | 在所属Actor位置移动时调用 |
    > UPrimitiveComponent的复杂在于其涉及到渲染，需要维护CPU、GPU两侧的状态，加载图元资源，调度着色器。

9.  AActor：任何**可放置**、可Spawn的类型的基类
    | 成员名称 | 成员类型 | 含义 | 注意 |
    | --- | --- | --- | --- |
    | AddActorWorldTransform() | 继承函数 | 用于对Actor做全局变换 | |
    | AddActorLocalRotation() | 继承函数 | 用于对Actor做局部坐标系旋转 | |
    | SetMaterial() | 继承函数 | 用于设置材质、材质实例 | |
    | SetCollisionEnabled() | 继承函数 | 用于设置碰撞计算方式 | |
    | AttachToComponent() | 函数 | 将当前Actor设置连接到指定Component | 常用于将某物品绑定到人物、其他物品身上 | |
    | Destroy() | 函数 | 删除当前Actor | |
    | SetLifeSpan() | 函数 | 设置当前Actor的剩余存活时间 | |
    | SetActorTickEnabled | 继承函数 | 用于开启、关闭Actor的Tick功能 | |
    | SetActorHiddenInGame | 继承函数 | 用于控制Actor的可见性（但不影响事件响应、可操作性） | |
    | NotifyActorBeginOverlap | 继承函数 | 用于提示和其他Actor重叠 | |
    | NotifyActorEndOverlap | 继承函数 | 用于提示和其他Actor结束重叠 | |
    | PostEditChangeProperty | 继承函数 | 用于接收并处理来自编辑器的成员属性修改事件 |  |
    | OnConstruction | 继承函数 | 属性变动时的回调 | 无法得知变动属性具体是哪个，注意性能 |

10. UWorld：世界类型
    | 成员名称 | 成员类型 | 含义 | 注意 |
    | --- | --- | --- | --- |
    | SpawnActor() | 泛型函数 | 创建一个Actor | |
    | DestroyActor() |  | 删除一个Actor | |
    | LineTraceSingleByObjectType | bool函数 | 以类型区分，计算射线首个命中物体 | 类型如ECC_Static、ECC_Pawn等 |

## 广泛继承的函数
| 名称 | 继承来源 | 含义 |
| --- | --- | --- |
| XXX::StaticClass() | | 创建并返回所属类型的静态实例 |
| GetWorld() | | 获取世界指针，常用于Spawn |
| GetComponentLocation（） | | 获取组件的位置 |

## 其他常用基本类型
| 名称 | 含义 | 注意 | 常用成员 |
| --- | --- | --- | --- |
| UStaticMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 静态网格体组件，即一个UStaticMesh的实例 | 可能被垃圾回收，需要用UPROPERTY进行标记 | SetupAttachment、SetStaticMesh、SetMobility |
| USkeletalMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 可动画化的网格体组件 | 也需用UPROPERTY标记 | SetAnimationMode、PlayAnimation、SetAnimInstanceClass、IsPlaying |
| UStaticMesh、USkeletalMesh | 静态、骨骼网格体底层存储类型 | 需设置给特定的component才能使用 | |
| FVector、FRotator | 向量、旋转子 | 常在SetXXXRotation/Location中使用 | |
| FString | 可变字符串 | 每个FString独立保存字符数组 | \*操作符获取保管字符串 |
| USpringArmComponent | 摇臂类，常用于辅助第三人称相机 | 一般绑定到角色 | bUsePawControlRotation |
| UCameraComponent | 相机类型 | 常绑定到摇臂 | bUsePawnControlRotation、GetForwardVector、 |
| UInputComponent | 输入组件，将输入事件绑定到具体函数 | | BindAxis、BindAction | 
| UCharacterMovementComponent | 角色运动组件 | 在角色类中用Get函数获取 | 各种用于控制角色移动的参数，IsFalling()等运动状态函数 |
| UAnimSequence | 动画序列 | 实际还是用动画蓝图更方便 | |
| UBoxComponent | 盒组件 | 常用于进行简单的碰撞检测 | InitBoxExtent、OnComponent(Begin/End)Overlap.AddDynamic设置碰撞检测回调 |
| UParticleSystemComponent | 粒子系统组件 | 常用于烟雾、火焰等 | SetTemplate |
| UPointLightComponent | 点光源组件 | | SetLightColor、SetIntensity、SetSourceXXXX |
| FHitResult | 命中结果类型 | 射线检测的结果类型 | Reset、GetActor、GetComponent |
| FCollisionQueryParams | 碰撞查询参数类型 | 用于射线检测等场景 | AddIgnoredActor |

## 常用工具类/枚举类/函数
| 名称 | 含义 | 注意 |
| --- | --- | --- |
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
| FAttachmentTransformRules | 连接时的变换规则枚举类 | 用于组件进行物理连接时，指定连接的运算方式 |
| DrawDebugLine/Mesh/... | 绘制一个调试用的线、面 | 用于调试 |
| FArguments | 参数包装类型 | 注意在不同类型中，都有这种子类型的定义 |
| GetHUD() | 获取HUD实例 | 在主程序中必须用该函数才能获取，而不能用GetWorld()->HUDClass |
| FLatentActionInfo | 延迟动作信息 | 用于登记延迟调用的函数的一些基础信息 |
| FDateTime | 获取时间信息 | |
| FModuleManager | 模块加载器 | 可以对模块进行加载判断、动态加载 |
| FActorSpawnParameters | 角色创建参数 | 配置对SpawnActor函数进行调节的参数 |


## UKismetSystemLibrary
- 一个丰富的类库，提供了从几何运算、绘制调试信息、延迟函数调用等多种不同种类的实用功能
    | 名称 | 含义 | 注意 |
    | --- | --- | --- |
    | Delay | 提供延迟运行 | 传递被调用的类对象实例、延迟时间、调用的函数信息封装 |


## 网络通信常用工具类/函数
| 名称 | 含义 | 注意 |
| --- | --- | --- |
| FHttpModule | HTTP模块 | 使用Get静态函数获取实例 |
| FHttpRequestRef | HTTP请求实例 | 由FHttpModule::CreateRequest()产生 |
| FHttpRequestCompleteDelegate | HTTP回调接口 | 用BindUObject来绑定具体的回调函数 |
| FJsonObjectConverter | Json实用工具 | |
| FJsonSerializer | Json序列化反序列化工具 | 分别操作TJsonReader、TJsonWriter |
| TJsonReaderFactory | JsonReader工厂 | 用Create成员函数将FString转为TJsonReader |
| FJsonObject | Json对象 | 是以哦那个GetXXXXField获取指定的Json成员 |
| UStructToJsonObjectString | 结构体转Json | FJsonObjectConverter静态函数 |
| FWebSocketsModule | WebSocket模块 | 实用Get获得实例 |
| IWebSocket | WebSocket接口 | 由FWebSocketModule::CreateWebSocket产生 |
| FWebSocketXXXEvent | IWebSocket中的各类事件接口 | 用AddUObject绑定具体回调 |

> 注1：类型系统中，所有非Class的内容，都是一种UObject。万物皆是UObject。

## 开发要点
1. 由于C++代码带来的变化不能像蓝图一样，自动显示在编辑器中，因此需要使用**Live Coding**功能，在不重启编辑器的情况下，编译并应用C++的罪行修改。快捷键是Ctrl+Alt+F11。
1. 常见的在项目设置中进行配置的内容：
    - 输入的事件绑定，按键事件、轴事件
    - 游戏模式
1. X轴正方向是默认的物体朝向、人脸朝向方向。因此在需要使用到面朝方向的计算时，也请使用GetUnitAxis(EAxis::X)。
1. 想要使用断点，需要在DebugGame Editor等模式下，以调试方式启动
1. 一些引擎特性在需要使用时，需要先在XXX.Build.cs中添加对应模块，比如：Slate、SlateCore、OnlineSubsystem、UMG


## 常见功能示例
1. 添加自定义C++类：    
    1. 步骤：菜单栏→工具→新建C++类→选择合适的父类并创建
1. 

## 一些无奈
1. UE类型体系内的用户自定义类型，必须以XXXX.generated.h为最后一个头文件。

## 模块
1. 说明：模块是UE组织软件架构的一种构建单元。适当的进行模块化的开发，提高软件开发、复用效率。
2. 为Editor添加自定义模块步骤
   1. 修改uproject文件，添加对自定义模块的依赖，形如
      ```json
      // ...
      "Modules": [
         {
            // ...
         },
         {
            "Name": "YourModule",
            "Type": "Editor",
            "LoadingPhase": "PostEngineInit",
            "AdditionalDependencies": [
               "Engine",
               "CoreUObject"
            ]
         }
         // ...
      ],
      ```
   2. 在Source文件夹下，新建以自定义模块为名称的文件夹，文件夹结构形如
      ```
         - （修改）YourProject.uproject
         - Source/
                  - YourProject/
                  -（新建）YouModule/
                           - YouModule.h & cpp
                           - YouModule.build.cs
                  -（修改）YourProject.Target.cs
                  - YourProjectEditor.Target.cs
      ```
   3. 在新建文件夹内，添加构建文件build.cs，形如
      ```cs
      using UnrealBuildTool;
      using System.Collections.Generic;

      public class YouModule : ModuleRules
      {
         public YouModule(ReadOnlyTargetRules Target) : base(Target)
         {
            PublicDependencyModuleNames.AddRange(new string[] {
                  "Core", "CoreUObject", "Engine", "InputCore","UnrealEd"});
            // 可选：模块依赖于主项目
            PublicDependencyModuleNames.Add("YourProject");
            PrivateDependencyModuleNames.AddRange(new string[] {"Core" });
         }
      }
      ```
   4. 添加.h/.cpp，形如
      ```cpp
      // h内
      #pragma once
      #include "CoreMinimal.h"
      #include "Modules/ModuleManager.h"

      class FDemoZeroModModule: public IModuleInterface
      {
      };
      
      // cpp内
      #include "DemoZeroMod.h"
      IMPLEMENT_MODULE(FDemoZeroModModule, DemoZeroMod)
      ```
   5. 修改Editor.cs，形如
      ```c#
      // Copyright Epic Games, Inc. All Rights Reserved.
      using UnrealBuildTool;
      using System.Collections.Generic;

      public class YourProjectEditorTarget : TargetRules
      {
         public YourProjectEditorTarget( TargetInfo Target) : base(Target)
         {
            Type = TargetType.Editor;
            DefaultBuildSettings = BuildSettingsVersion.V2;
            IncludeOrderVersion = EngineIncludeOrderVersion.Unreal5_1;
            ExtraModuleNames.Add("YourProject");
            ExtraModuleNames.Add("YourModule");
         }
      }
      ```
   > 注：Module开发容易受UE引擎版本影响，注意对依赖模块的使用

   > 参考：
   > - [UE5.2 Unreal Engine Modules](https://docs.unrealengine.com/5.2/en-US/unreal-engine-modules/)
   > - [UE5.2 UBT Target](https://docs.unrealengine.com/5.2/en-US/unreal-engine-build-tool-target-reference/)
3. 常见API
   | 名称 | 所属类 | 类型 | 说明 |
   | --- | --- | --- | --- |
   | StartupModule | IModuleInterface | 继承函数 | 重写该函数以在Module加载后执行所需代码 |
   | ShutdownModule | IModuleInterface | 继承函数 | 重写该函数以在Module卸载后执行所需代码 |

4. 注意事项：
   1. Module开发不支持热更新，需要关闭编辑器，编译运行

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
1. 回调函数必须添加宏描述UFUNCTION()，例如碰撞检测回调（此举实际上将其加入反射系统）
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
1. CreateDefaultSubobject函数只能在构造函数内调用，不能在BeginPlay等函数中使用（包括间接使用）。想要动态创建，只能使用NewObject，并进一步使用AttachTo等方式绑定父组件。

## 参考
1. [【虚幻5】【不适合小白观看】用C++来进行基于UE5的游戏开发（含动画蓝图）](https://www.bilibili.com/video/BV17Q4y1Y7fr)
2. [官网：C++编程 虚幻引擎编程开发的相关信息](https://docs.unrealengine.com/5.0/zh-CN/programming-with-cplusplus-in-unreal-engine/)
3. [官网：Slate UI编程](https://docs.unrealengine.com/5.0/zh-CN/slate-user-interface-programming-framework-for-unreal-engine/)
4. [UE5-c++教程 01~05](https://www.bilibili.com/video/BV1be41137Kp)
5. [知乎专栏：UE从点Play开始](https://zhuanlan.zhihu.com/p/512249255)
6. [Unreal Engine C++ Advanced Dark Souls Boss Fight System](https://www.youtube.com/watch?v=ANzEGECpd0g)
7. [UE5 C++ Tutorial | Introduction to Unreal Engine 5 with C++ in less than 90 Minutes](https://www.youtube.com/watch?v=nvruYLgjKkk&list=PL-m4pn2uJvXHL5rxdudkhqrSRM5gN43YN)
8. [UE4静态/动态加载资源方式](https://zhuanlan.zhihu.com/p/266859719)
9.  [【教程】虚幻5教程 斯坦福专用课程 UE4 & C++ 专业游戏开发教程 24.5小时 中文字幕](https://www.bilibili.com/video/BV1nU4y1X7iQ)
10. [UE4 UCLASS宏和可用宏参数](https://zhuanlan.zhihu.com/p/148098617)
11. [UE5.1 Networking Overview](https://docs.unrealengine.com/5.1/en-US/networking-overview-for-unreal-engine/)
12. [UE5.2 编程和脚本编写](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-programming-and-scripting/)
13. 《Unreal Engine4 Scripting with C++ Cookbook》