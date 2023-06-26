---
title: "UE5学习：Cpp引擎核心篇（一）"
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
   - 作用：在UBT确定了需要更新的编译单元后开始工作，为指定的**头文件**添加反射等所需的各类代码，每个编译单元最终生成一个.inl文件。
   - 注意：
        1. 由于UHT并没有具备完整的C++解析能力，因此除了```WITH_EDITOR``` / ```WITH_EDITORONLY_DATA```之外，应当尽量避免在UPROPERTY等宏附近使用条件编译```#ifdef```。
        2. 受UHT限制，无法在头文件中对使用了```UPROPERTY```等宏标记的变量，再使用```using```、```typedef```等方式定义的类型。
        3. 受UHT限制，不能对重载函数（overload）使用UFUNCTION，对重写函数（override）使用UFUNCTION时，一般都是对```UInterface```中的函数进行重载，同时要求必须添加C++的virtual标记，或者添加宏标记```BlueprintImplementableEvent```。

### 委托
1. 概述：UE中委托和代理广泛存在，这两种模式在事件、渲染等机制中被广泛使用
2. 委托（delegate）步骤：
   1. 核心宏：用于描述委托函数元属性的```UDELEGATE()```、用于描述委托函数的函数签名的```DECLARE_XXX_DELEGATE_XXX```。
   2. 使用宏定义委托函数，如```MyActionDelegate```
   3. 在必要时进行绑定```MyActionDelegate.Bind()```，以及委托调用```MyActionDelegate.Execute()```
   > 事件（event）：事件定义宏```DECLARE_EVENT()```只是一种特殊的多播委托，对```Broadcast()```、```Bind()```、```Execute()```等做了权限约束，不允许非持有对象调用

## 其他核心类型

### Actor
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

### Component
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
3. RootComponent：
   1. 特点：包含变换数据，位置、旋转，缩放。当一个```Actor```没有任何```RootComponent```时，则不具备这些数据。
   2. 所有其他的```Component```都必须以某种方式，绑定到```RootComponent```

### Interface
1. 基本说明：和接口的含义一样，用来解决类型体系中，需要提供相同功能，但是不具备"Is A"关系的类型情况。如果直接多继承UObject，会导致严重的问题（比如存在多个UClass实例，难以区分）。总而言之，用接口来实现"Has A"是一个很好的办法。
   > 需要用Interface的场景主要是：所属类型不在Actor类型体系下，因此无法使用ActorComponent来完成一些接口定义。此时使用Interface。
2. 基本步骤
   1. 编辑器自动：定义一个```UINTERFACE```标记的```UInterface```子类，该类不需修改，也不应添加任何东西，定义一个同名但前缀为```I```原始C++类。
      1. ```UInterface```可以继承自```UObject```或者其他```UInterface```。
   2. 编写所需要的函数，如果需要禁止默认行为可以在其定义中添加```unimplemented()```。
   3. 在所需场合继承前缀为```I```的原始类，并重写对应函数。
3. 相关代码工具
   1. 判断是否是一个接口的实现
      ```cpp
      // 使用U前缀和I前缀，成对的定义接口的意义，依然可以一定程度上使用反射系统
      // 这里的UMyInterface代表了你定义的被UINTERFACE()宏修饰的接口名称
      UClass::ImplementsInterface(UMyInterface::StaticClass())
      ```
   2. 转型
      ```cpp
      // 转型仍使用Cast，但是需要使用I前缀，即原始C++类
      IMyInterface* temp = Cast<IMyInterface>(obj);
      if(temp)
      {
         // ...
      }
      ```
   3. 暴露给蓝图：只需给```I```前缀的原始类中的函数添加```UFUNCTION()```宏即可
      ```cpp
      UFUNCTION(BlueprintCallable, Category = Debug)
      virtual void MyOnPostBeginPlay()
      {
         // ...
      }
      ```
   4. 用C++调用蓝图中实现的接口（和3相反）
      ```cpp

      ```
4. 注意：
   1. 以```I```为前缀的类，是原始C++类，不是```UObject```。因此涉及到反射时用```U```前缀，其他情况用另一种。
5. 代码示例
```cpp
#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "AnimPlayInterface.generated.h"

// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class UAnimPlayInterface : public UInterface
{
	GENERATED_BODY()
};

/**
 */
class DEMOZERO_API IAnimPlayInterface
{
	GENERATED_BODY()
	// Add interface functions to this class. 
   // This is the class that will be inherited to implement this interface.
public:
   virtual void Play() {
      // 如果希望默认实现被调用会报错，可以标记未完成
      unimplemented();
   }
};
```

## 编辑器环境Editor
1. 概述：在UE自身提供的编辑器环境之外，UE也允许玩家对编辑器环境进行改变，编写一些自定义模块、代码，在编辑器环境下工作，以提升开发效率。Editor开发中普遍使用Slate
2. Editor常见类型
   | 名称 | 类型 | 说明 |
   | --- | --- | --- |
   | TCommands | 类模板 | 使用CRTP写法，完成UI和具体动作的解耦 |
   | UI_COMMAND | 宏 | 创建一个可执行命令的UI控件 |
   | IMainFrameModule | 接口类 |  |
   | FLevelEditorModule | 工具类 | 关卡编辑器模块 |
   | FModuleManager | 工具类 | 对模块加载进行管理 |
   | FUICommandInfo | 工具类 | 存储UI相关命令信息（描述、名称等） |
   | FUICommandList | 工具类 | 存储一系列UI命令信息 |
   | FExtender | 工具类 | 用于向UI添加新控件 |
   | FEditorStyle | 工具类 | 内含控制Slate的UI风格的各种属性 |
   | EUserInterfaceActionType | 枚举类 | 表明控件的类型（按钮，开关，列表等） |

   > 注：TCommands使用过程中，利用了C++的CRTP特性（Curious Recurring Template Pattern），用来实现静态多态。参考[C++ 惯用法 CRTP 简介](https://liam.page/2016/11/26/Introduction-to-CRTP-in-Cpp/)。
### 自定义UI流程
1. 工具栏按钮
   1. 注册命令、查找目标窗口、在目标窗口中添加命令按钮
2. 

## 游戏运行时框架
> GamePlay Framework

### 帧


## 其他子系统
### 网络通信
1. 概述：UE内部实现了TCP、UDP、HTTP、Websocket等协议。但如果打算使用UE自带的Dedicated Server进行服务器端开发，需要自行从源代码编译UE。
2. 原理：

### 蓝图
1. C++和蓝图的互操作的各种需求
   1. C++函数希望暴露函数给蓝图：宏参数```BlueprintCallable```
   2. C++函数希望由蓝图实现：宏参数```BlueprintImplementableEvent```
   3. C++函数给出默认实现，但蓝图仍然可以重写：宏参数```BlueprintNativeEvent```，注意此时C++的默认实现，带有后缀```_Implementation```，如果蓝图未给出实现，则自动生成的函数将会调用带有该后缀的版本。
      ```cpp
      // MyInterface.h
      class XXX_API IMyInterface
      {
         GENERATED_BODY()
      public:
         // 不用实现
         void DoSomething();

         // 给出默认实现
         void DoSomething_Implementation();
      }

      // 判断是否实现接口，并通用的执行（由运行时决定调用蓝图还是C++默认实现）
      if(MyActor->GetClass()->ImplementsInterface(UMyInterface::StaticClass()))
      {
         IMyInterface::Execute_DoSomething();
      }
      ```
   4. ```OnConstruction```：C++侧，对蓝图内```Construction Script```(任意属性变动时会执行)的实现函数
2. 默认情况
   1. 出于性能考虑（减少反射相关代码量），C++的变量、类型默认并不会导出到蓝图中。
   > 尽量不要依赖于默认行为

## 其他参考
1. [知乎专栏：InsideUE系列](https://www.zhihu.com/column/insideue4)
2. [Unreal Property System Reflection](https://www.unrealengine.com/zh-CN/blog/unreal-property-system-reflection)