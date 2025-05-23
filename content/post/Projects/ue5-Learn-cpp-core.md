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
draft: true
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
        2. 受UHT限制
           1. 无法在头文件中对使用了```UPROPERTY```等宏标记的变量，再使用```using```、```typedef```等方式定义的类型。
           2. 不能对重载函数（overload）使用```UFUNCTION```
           3. 对重写函数（override）使用```UFUNCTION```时，一般都是对```UInterface```中的函数进行重载，同时要求必须添加C++的virtual标记，或者添加宏标记```BlueprintImplementableEvent```，且派生类重写时，不许再添加```UFUNCTION```（保持反射属性）

### 非UObject系统
1. 以F为前缀的类型
   - 特点：主要是工具类，例如```FAssetTypeActions_Base```、```FGraphPanelPinFactory```。也包括大部分结构体，如```FColor```、```FVector```。
   - 注意：不能使用```UCLASS```等宏进行标记（标记也会报错），也没有必要，这些类型不需要经过UHT处理。
1. 以S为前缀的类型
   - 特点：代表Slate类型，如```SWidget```
   - 注意：也不能使用```UCLASS```等宏进行标记，但需要使用```SLATE_BEGIN_ARGS```、```SLATE_END_ARGS```等Slate宏，完善UI代码。
1. 以I为前缀的类型
   - 特点：是接口类型，可能是```UINTERFACE```的伴生类型，也可能就是独立的一个接口。该前缀类型的派生类型，并不需要以I为前缀，视其使用目的，可以使用F前缀。
   - 注意：重写其虚函数以满足业务和接口要求。

> [推荐的资源命名惯例Recommended Asset Naming Conventions](https://docs.unrealengine.com/5.2/en-US/recommended-asset-naming-conventions-in-unreal-engine-projects/)

### 委托
1. 概述：UE中委托（delegate）和代理广泛存在，这两种模式在事件、渲染等机制中被广泛使用
2. 成员函数作为委托的步骤：
   1. 核心宏：用于描述委托函数元属性的```UDELEGATE()```、用于描述委托函数的函数签名的```DECLARE_XXX_DELEGATE_XXX```。
   2. 使用宏定义委托函数，如```MyActionDelegate```
   3. 在必要时进行绑定```MyActionDelegate.Bind()```，以及委托调用```MyActionDelegate.Execute()```
   > 事件（event）：事件定义宏```DECLARE_EVENT()```只是一种特殊的多播委托，对```Broadcast()```、```Bind()```、```Execute()```等做了权限约束，不允许非持有对象调用
3. 创建一个可用于绑定的委托函数实例：
   1. ```CreateWeakLambda()```：绑定弱引用对象的委托函数
   2. ```CreateRaw()```：绑定原始（非UFUNCTION）成员函数
   3. ```CreateSP()```：绑定共享指针对象的委托函数
   4. ```CreateUFunction()```：绑定UFUNCTION作为委托函数
   5. ```CreateUObject()```：绑定指定UObject的委托函数
   6. ```CreateStatic()```：使用全局函数，创建一个静态委托函数
   7. ```CreateLambda()```：很多类型提供委托工具类，可以直接创建Lambda函数作为委托。
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
1. 概述：在UE自身提供的编辑器环境之外，UE也允许玩家对编辑器环境进行改变，编写一些自定义模块、代码，在编辑器环境下工作，以提升开发效率。Editor开发中普遍使用Slate。
   > 参考
   > - [UE 插件与工具开发：基础概念](https://imzlp.com/posts/75405/)
2. Editor开发常见类型

      | 名称 | 类型 | 说明 |
      | --- | --- | --- |
      | IMPLEMENT_MODULE| 宏 | 必备，用于导出该模块实例 |
      | TCommands | 类模板 | 使用CRTP写法，完成UI和具体动作的解耦 |
      | UI_COMMAND | 宏 | 创建一个可执行命令的UI控件 |
      | IMainFrameModule | 接口类 | 用于管理Editor最顶层的主窗口 |
      | FLevelEditorModule | 工具类 | 关卡编辑器模块 |
      | FModuleManager | 工具类 | 对模块加载进行管理 |
      | FUICommandInfo | 工具类 | 存储UI相关命令信息（描述、名称等） |
      | FUICommandList | 工具类 | 存储一系列UI命令信息 |
      | FExecuteAction | 工具类 | 创建一个可以和控件绑定的回调函数委托 |
      | FUIAction | 工具类 | UI动作的实现，通常用于接收一个回调委托 |
      | FExtender | 工具类 | 用于向UI添加新控件，工具栏、菜单项等均可 |
      | FEditorStyle | 工具类 | 内含控制Slate的UI风格的各种属性 |
      | EUserInterfaceActionType | 枚举类 | 表明控件的类型（按钮，开关，列表等） |
      | FMenuBuilder | 工具类 | 构造菜单 |
      | FToolBarBuilder | 工具类 | 构造工具栏 |
      | FToolBarExtensionDelegate | 工具类 | 用于添加自定义ToolbarExtender |
      | FMenuExtensionDelegate | 工具类 | 用于添加自定义MenuExtender |
      | FSlateApplication | 工具类 | Slate应用程序，有很多静态成员 |
      | IAssetTools | Asset工具接口 | 用于自定义资源管理菜单等 |
      | FAssetToolsModule | 资源工具模块 | 用于管理资源类型，添加自定义资源类 |
      | IConsoleCommand | 控制台命令接口 | 用于管理控制台命令 |
      | IConsoleManager | 控制台管理器接口 | 用于获取、管理控制台实例 |
      | FConsoleCommandDelegate | 控制台命令委托工具类 |  用于创建控制台命令 |
      | FConsoleCommandWithArgsDelegate | 控制台命令委托工具类 | 用于创建带参数控制台命令的Lambda回调函数 |
      | SGraphPin | 蓝图节点子控件类 | 蓝图节点图内的各种可操作控件都叫做Pin |
      | FGraphPanelPinFactory | 蓝图节点子控件工厂 | 继承以重写工厂函数，自定义蓝图节点子控件 |
      | FEdGraphUtilities | 编辑器图表工具实例 | 可用于注册蓝图节点工厂 |
      | IDetailCustomization | 细节面板接口类 | 提供对细节面板的控制 |
      | IDetailLayoutBuilder | 细节面板布局构造类 | 用于对细节面板布局进行管理 |
      | FPropertyEditorModule | 属性编辑器模块 | 用于注册自定义的属性细节面板 |
      > 注：TCommands使用过程中，利用了C++的CRTP特性（Curious Recurring Template Pattern），用来实现静态多态。参考[C++ 惯用法 CRTP 简介](https://liam.page/2016/11/26/Introduction-to-CRTP-in-Cpp/)。

      > 这里列出的只是冰山一角，而且在使用过程中，大概率需要引入中间变量的一些头文件，总之开发时要多看文档。
3. 坑
   1. 模块加载和卸载顺序：虽然官方说了，为了保证依赖不出问题，加载顺序和卸载顺序相反。但是，实际上有些模块被优先卸载了（说是为了禁止在卸载期间仍然调用这些模块）。在5.2版中这些模块有：```AssetTools```、```WorldBrowser```、```AssetRegistry```、```IPackageResourceManager```、```IBulkDataRegistry```、```FShaderPipelineCache```、```FShaderCodeLibrary```、```FIoDispatcher```等。如果想要对依赖这些的模块做完美卸载，需要额外想办法。
### 自定义UI流程
1. 模块基本流程
   1. 准备工作：编写自定义命令类型，重写命令注册函数，编写命令回调函数
   2. StartupModule阶段：注册命令、映射命令和回调函数、查找或创建目标窗口、在目标窗口的合适位置添加命令相关UI控件。
   3. ShutdownModule阶段：从目标窗口中移除命令相关UI控件
   > Editor界面中有若干个被称为扩展点（UIExtension Point）的地方，根据不同的类型，可以向其前后扩展自定义控件。
2. 添加新的Asset类型
   1. 继承```UObject```并创建Asset类型，并为其创建工厂类型（继承```UFactory```），重写```FactoryCreateNew```函数。
   2. 进一步地，可以为自定义Asset类型添加专属的右键菜单项，以提供专属功能。继承```FAssetTypeActions_Base```，重写必要的虚函数，如```HasActions```，```GetActions```等。此时已经可以在创建菜单、内容浏览器等位置，看到自定义Asset类型的标志
   3. 在```StartupModule```阶段使用```FModuleManager```获取AssetTools，并注册自定义Asset类型的菜单项动作。注意```ShutdownModule```阶段也应该取消注册。
   > 参考：[Creating a Custom Asset Type with its own Editor in C++](https://dev.epicgames.com/community/learning/tutorials/vyKB/unreal-engine-creating-a-custom-asset-type-with-its-own-editor-in-c)
3. 为自定义Asset定制蓝图节点UI
   1. 继承```FGraphPanelPinFactory```，重写```CreatePin```虚函数，内部也是调用Slate的SNew创建UI。
   2. 继承```SGraphPin```，该类需使用Slate框架的```SLATE_BEGIN_ARGS```、```SLATE_END_ARGS```宏等，来给出一个标准的Slate控件类的实现。
   3. StartupModule阶段：实例化自定义Pin的工厂，注册到蓝图工具中。
   > 默认的Asset类型，在蓝图中是以UObject的形式进行显示，对于其属性并不会优化显示、编辑方式，对于有需要的情况，可以自定义其在蓝图内的编辑方式。比如对FColor类型的成员，在蓝图节点内提供一个取色器，而不是手动输入ARGB。
4. 自定义细节面板
   1. 准备工作：继承```IDetailCustomization```，重写```CustomizeDetails```虚函数。向```DetailBuilder```中添加自定义的Slate控件
   2. StartupModule阶段：获取属性编辑模块，向其添加自定义Asset及其自定义细节面板。
   3. ShutdownModule阶段：取消注册。
5. 添加新的命令行命令
   1. 准备工作：编辑自定义命令行命令的执行回调函数
   2. StartupModule阶段：获取控制台实例并向其注册控制台命令
   3. ShutdownModule阶段：取消注册
6. 一些Editor中的开发辅助工具
   1. Pick Live Widget：UE5.2内置了Slate UI的调试工具，在 **Tools$\to$Debug$\to$Widget Reflector** 中，详情参考[Widget Reflector](https://docs.unrealengine.com/5.2/en-US/using-the-slate-widget-reflector-in-unreal-engine/)。允许动态的查看所有Slate组件的层级关系。
   2. 编辑器 **偏好设置$\to$显示UI扩展点** ：Display UIExtension Point，能够显示允许扩展的位置的名称，便于在各种```Extender```中选择插入点。
### 代码示例
- <details><summary>自定义模块 DemoZeroMod.h/cpp</summary>

   ```cpp
   // .h
   class FDemoZeroModModule: public IModuleInterface
   {
   public:
      // 在虚函数中重写自定义模块的注册等业务
      virtual void StartupModule() override;
      virtual void ShutdownModule() override;

      // 各类资源
      TSharedPtr<FExtender> ToolbarExtender;
      TSharedPtr<const FExtensionBase> Extension;
      TSharedPtr<const FExtensionBase> MenuExtension;
      TSharedPtr<FAssetTypeActions_Base> Actions;
      TSharedPtr<FModAssetGraphPinFactory> PinFactory;

      IConsoleCommand* DisplayTestCommand;
      IConsoleCommand* DisplayUserSpecifiedWindow;

      void MyButton_Clicked();
      void AddToolbarExtension(FToolBarBuilder& builder);
      void AddMenuExtension(FMenuBuilder& builder);
      void DisplayWindow(FString args);
   };

   // .cpp
   #include "DemoZeroMod.h"
   #include "Widgets/SWindow.h"
   #include "LevelEditor.h"
   #include "Interfaces/IMainFrameModule.h"
   #include "AssetToolsModule.h"
   #include "ModAssetAction.h"
   #include "ModAsset.h"
   #include "ModAssetDetailCustomization.h"
   #include "DemoZeroModCommands.h"

   IMPLEMENT_MODULE(FDemoZeroModModule, DemoZeroMod)

   void FDemoZeroModModule::StartupModule()
   {
      UE_LOG(LogTemp, Log, TEXT("FDemoZeroModModule Startup!"));

      // 自定义命令注册
      FDemoZeroModCommands::Register();
      TSharedPtr<FUICommandList> CommandList =
         MakeShareable(new FUICommandList());
      // 绑定自定义命令的输入控件和回调委托
      CommandList->MapAction(FDemoZeroModCommands::Get().MyButton,
         FExecuteAction::CreateRaw(this,
            &FDemoZeroModModule::MyButton_Clicked),
         FCanExecuteAction());

      // 添加Toolbar扩展器
      ToolbarExtender = MakeShareable(new FExtender());
      // 添加到Play扩展点之后
      Extension = ToolbarExtender
         ->AddToolBarExtension("Play", EExtensionHook::Before,
            CommandList, FToolBarExtensionDelegate::CreateRaw(this,
               &FDemoZeroModModule::AddToolbarExtension));
      // 将Toolbar扩展器，添加到LevelEditor模块中
      FLevelEditorModule& LevelEditorModule =
         FModuleManager::LoadModuleChecked<FLevelEditorModule>("LevelEditor");
      LevelEditorModule.GetToolBarExtensibilityManager()->AddExtender(ToolbarExtender);

      // 添加MenuEntry
      MenuExtension = ToolbarExtender->AddMenuExtension("LevelEditor", EExtensionHook::Before,
            CommandList, FMenuExtensionDelegate::CreateRaw(this,
               &FDemoZeroModModule::AddMenuExtension));
      LevelEditorModule.GetMenuExtensibilityManager()->AddExtender(ToolbarExtender);

      // 注册ModAsset
      IAssetTools& AssetTools =
         FModuleManager::LoadModuleChecked<FAssetToolsModule>("AssetTools").Get();
      Actions = MakeShareable(new FModAssetAction);
      AssetTools.RegisterAssetTypeActions(Actions.ToSharedRef());

      // 添加自定义控制台命令
      DisplayTestCommand =
         IConsoleManager::Get().RegisterConsoleCommand(TEXT("DisplayTestCommandWindow"), TEXT("test"),
            FConsoleCommandDelegate::CreateRaw(this,
               &FDemoZeroModModule::DisplayWindow,
               FString(TEXT("Test Command Window"))), ECVF_Default);
      DisplayUserSpecifiedWindow =
         IConsoleManager::Get().RegisterConsoleCommand(TEXT("DisplayWindow"), TEXT("test"),
            FConsoleCommandWithArgsDelegate::CreateLambda(
               [&](const TArray< FString >& Args)
               {
                  FString WindowTitle;
                  for (FString Arg : Args)
                  {
                     WindowTitle += Arg;
                     WindowTitle.AppendChar(' ');
                  }
                  this->DisplayWindow(WindowTitle);
               }
      ), ECVF_Default);

      // 自定义蓝图节点UI
      PinFactory = MakeShareable(new FModAssetGraphPinFactory);
      FEdGraphUtilities::RegisterVisualPinFactory(PinFactory);

      // 细节面板
      FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
      PropertyModule.RegisterCustomClassLayout(UModAsset::StaticClass()->GetFName(),
               FOnGetDetailCustomizationInstance::CreateStatic(&FModAssetDetailCustomization::MakeInstance));
   }

   void FDemoZeroModModule::ShutdownModule()
   {
      UE_LOG(LogTemp, Log, TEXT("FDemoZeroModModule Shutdown!"));
      // 尚未解决：AssetTools在卸载阶段优先于大部分模组，无法通过这种方式安全卸载自定义Actions
      //IAssetTools& AssetTools =
      //	FModuleManager::LoadModuleChecked<FAssetToolsModule>("AssetTools").Get();
      //AssetTools.UnregisterAssetTypeActions(Actions.ToSharedRef());
      Actions.Reset();

      ToolbarExtender->RemoveExtension(Extension.ToSharedRef());
      Extension.Reset();
      MenuExtension.Reset();
      ToolbarExtender.Reset();
      FDemoZeroModCommands::Unregister();


      IConsoleManager::Get().UnregisterConsoleObject(TEXT("DisplayTestCommandWindow"));
      
      IConsoleManager::Get().UnregisterConsoleObject(TEXT("DisplayWindow"));

      FEdGraphUtilities::UnregisterVisualPinFactory(PinFactory);
      PinFactory.Reset();

      FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
      PropertyModule.UnregisterCustomClassLayout(UModAsset::StaticClass()->GetFName());
   }

   void FDemoZeroModModule::MyButton_Clicked()
   {
      TSharedRef<SWindow> CookbookWindow = SNew(SWindow)
         .Title(FText::FromString(TEXT("DemoZero Window")))
         .ClientSize(FVector2D(800, 400))
         .SupportsMaximize(false)
         .SupportsMinimize(false)
         [
            SNew(SVerticalBox)
            +SVerticalBox::Slot()
            .HAlign(HAlign_Center)
            .VAlign(VAlign_Center)
            [
               SNew(STextBlock)
               .Text(FText::FromString(TEXT("Hello from DemoZero Mod")))
            ]
         ];
      IMainFrameModule& MainFrameModule =
         FModuleManager::LoadModuleChecked<IMainFrameModule>(TEXT("MainFrame"));
      if (MainFrameModule.GetParentWindow().IsValid())
      {
         FSlateApplication::Get().AddWindowAsNativeChild
         (CookbookWindow, MainFrameModule.GetParentWindow()
            .ToSharedRef());
      }
      else
      {
         FSlateApplication::Get().AddWindow(CookbookWindow);
      }
   }

   void FDemoZeroModModule::AddToolbarExtension(FToolBarBuilder& builder)
   {
      FSlateIcon IconBrush =
         FSlateIcon(FEditorStyle::GetStyleSetName(),
            "LevelEditor.ViewOptions",
            "LevelEditor.ViewOptions.Small");
      builder.AddToolBarButton(FDemoZeroModCommands::Get()
         .MyButton, NAME_None, FText::FromString("My Button"),
         FText::FromString("Click me to display a message"),
         IconBrush, NAME_None);
   }

   void FDemoZeroModModule::AddMenuExtension(FMenuBuilder& builder)
   {
      FSlateIcon IconBrush =
         FSlateIcon(FEditorStyle::GetStyleSetName(),
            "LevelEditor.ViewOptions",
            "LevelEditor.ViewOptions.Small");
      builder.AddMenuEntry(FDemoZeroModCommands::Get().MyButton);
   }

   void FDemoZeroModModule::DisplayWindow(FString args)
   {
      TSharedRef<SWindow> CookbookWindow = SNew(SWindow)
         .Title(FText::FromString(TEXT("DemoZero Window")))
         .ClientSize(FVector2D(800, 400))
         .SupportsMaximize(false)
         .SupportsMinimize(false)
         [
            SNew(SVerticalBox)
            + SVerticalBox::Slot()
         .HAlign(HAlign_Center)
         .VAlign(VAlign_Center)
         [
            SNew(STextBlock)
            .Text(FText::FromString(args))
         ]
         ];
      IMainFrameModule& MainFrameModule =
         FModuleManager::LoadModuleChecked<IMainFrameModule>
         (TEXT("MainFrame"));
      if (MainFrameModule.GetParentWindow().IsValid())
      {
         FSlateApplication::Get().AddWindowAsNativeChild
         (CookbookWindow, MainFrameModule.GetParentWindow()
            .ToSharedRef());
      }
      else
      {
         FSlateApplication::Get().AddWindow(CookbookWindow);
      }
   }

   ```

  </details>
- <details><summary>自定义资源 ModAsset.h</summary>

  ```cpp
   #pragma once

   #include "CoreMinimal.h"
   #include "UObject/NoExportTypes.h"
   #include "ModAsset.generated.h"

   /**
   * 
   */
   UCLASS(BlueprintType)
   class DEMOZEROMOD_API UModAsset : public UObject
   {
      GENERATED_BODY()
   public:
      UPROPERTY(EditAnywhere, Category = "Custom Asset")
      FString ModName;

      UPROPERTY(EditAnywhere, Category = "Custom Asset")
      FString ModColor;
   };

   // 暂时不需要cpp内容
  ```

  </details>
- <details><summary>自定义资源动作 ModAssetAction.h/cpp</summary>
  
   ```cpp
   //h
   #pragma once

   #include "CoreMinimal.h"
   #include "AssetTypeActions_Base.h" 	
   #include "Framework/MultiBox/MultiBoxBuilder.h"
   /**
   * 
   */
   class DEMOZEROMOD_API FModAssetAction: public FAssetTypeActions_Base
   {
   public:
      virtual FText GetName() const override;
      virtual UClass* GetSupportedClass() const override;
      virtual FColor GetTypeColor() const override;
      virtual uint32 GetCategories() override;
      virtual void GetActions(const TArray<UObject*>& InObjects, FMenuBuilder& MenuBuilder) override;
      virtual bool HasActions(const TArray<UObject*>& InObjects)const override;

      void ModAssetContext_Clicked();
   };

   // cpp
   #include "ModAssetAction.h"
   #include "ModAsset.h" 	
   #include "EditorStyleSet.h"
   #include "Interfaces/IMainFrameModule.h"

   FText FModAssetAction::GetName() const
   {
      return FText::FromString(TEXT("Mod Asset"));
   }

   UClass* FModAssetAction::GetSupportedClass() const
   {
      return UModAsset::StaticClass();;
   }

   FColor FModAssetAction::GetTypeColor() const
   {
      return FColor::Green;
   }

   uint32 FModAssetAction::GetCategories()
   {
      return EAssetTypeCategories::Misc;
   }

   void FModAssetAction::GetActions(const TArray<UObject*>& InObjects, FMenuBuilder& MenuBuilder)
   {
      MenuBuilder.AddMenuEntry(
         FText::FromString("ModAssetAction"),
         FText::FromString("Action from Test"),
         FSlateIcon(FEditorStyle::GetStyleSetName(),
            "LevelEditor.ViewOptions"),
         FUIAction(
            FExecuteAction::CreateRaw(this,
               &FModAssetAction::ModAssetContext_Clicked),
            FCanExecuteAction()));
   }

   bool FModAssetAction::HasActions(const TArray<UObject*>& InObjects) const
   {
      return true;
   }

   void FModAssetAction::ModAssetContext_Clicked()
   {
      TSharedRef<SWindow> CookbookWindow = SNew(SWindow)
         .Title(FText::FromString(TEXT("Cookbook Window")))
         .ClientSize(FVector2D(800, 400))
         .SupportsMaximize(false)
         .SupportsMinimize(false);
      IMainFrameModule& MainFrameModule = FModuleManager::LoadModuleChecked<IMainFrameModule>(TEXT("MainFrame"));
      if (MainFrameModule.GetParentWindow().IsValid())
      {
         FSlateApplication::Get().AddWindowAsNativeChild
         (CookbookWindow, MainFrameModule.GetParentWindow()
            .ToSharedRef());
      }
      else
      {
         FSlateApplication::Get().AddWindow(CookbookWindow);
      }
   }
   ```

  </details>
- <details><summary>自定义资源工厂 ModAssetFactory.h/cpp</summary>

   ```cpp
   #pragma once

   #include "CoreMinimal.h"
   #include "Factories/Factory.h"
   #include "ModAssetFactory.generated.h"

   /**
   * 
   */
   UCLASS()
   class DEMOZEROMOD_API UModAssetFactory : public UFactory
   {
      GENERATED_BODY()
   public:
      UModAssetFactory();

      virtual UObject* FactoryCreateNew(UClass* InClass, UObject* InParent, FName InName, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn, FName CallingContext) override;
   };

   // cpp
   #include "ModAssetFactory.h"
   #include "ModAsset.h"

   UModAssetFactory::UModAssetFactory()
   {
      bCreateNew = true;
      bEditAfterNew = true;
      SupportedClass = UModAsset::StaticClass();
   }

   UObject* UModAssetFactory::FactoryCreateNew(UClass* InClass, UObject* InParent, FName InName, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn, FName CallingContext)
   {
      return NewObject<UModAsset>(InParent,InClass,InName,Flags);
   }

   ```

  </details>
- <details><summary>自定义资源细节面板 ModAssetDetailCustomization.h/cpp</summary>

   ```cpp
   #pragma once

   #include "CoreMinimal.h"
   #include "IDetailCustomization.h"
   #include "DetailLayoutBuilder.h"
   #include "ModAsset.h"
   //#include "IPropertyTypeCustomization.h"

   /**
   * 
   */
   class DEMOZEROMOD_API FModAssetDetailCustomization:public IDetailCustomization
   {
   public:
      virtual void CustomizeDetails(IDetailLayoutBuilder& DetailBuilder) override;
      void ColorPicked(FLinearColor SelectedColor);

      static TSharedRef<IDetailCustomization> MakeInstance()
      {
         return MakeShareable(new FModAssetDetailCustomization);
      }

      TWeakObjectPtr<UModAsset> MyAsset;
   };

   // cpp
   #include "ModAssetDetailCustomization.h"
   #include "Runtime/AppFramework/Public/Widgets/Colors/SColorPicker.h"
   #include "IDetailsView.h"
   #include "DetailCategoryBuilder.h"
   #include "Widgets/SBoxPanel.h" 	
   #include "DetailWidgetRow.h"
   //#include "PropertyEditor/Public/DetailLayoutBuilder.h"

   void FModAssetDetailCustomization::CustomizeDetails(IDetailLayoutBuilder& DetailBuilder)
   {
      const TArray< TWeakObjectPtr<UObject>>& SelectedObjects = DetailBuilder.GetDetailsView()->GetSelectedObjects();
      for (int32 ObjectIndex = 0; !MyAsset.IsValid() &&
         ObjectIndex < SelectedObjects.Num(); ++ObjectIndex)
      {
         const TWeakObjectPtr<UObject>& CurrentObject =
            SelectedObjects[ObjectIndex];
         if (CurrentObject.IsValid())
         {
            MyAsset = Cast<UModAsset>(CurrentObject.Get());
         }
      }
      DetailBuilder.EditCategory("CustomCategory", FText::GetEmpty(), ECategoryPriority::Important)
         .AddCustomRow(FText::GetEmpty())
         [
            SNew(SVerticalBox) + SVerticalBox::Slot().VAlign(VAlign_Center)
            [
               SNew(SColorPicker).OnColorCommitted(this, &FModAssetDetailCustomization::ColorPicked)
            ]
         ];
   }

   void FModAssetDetailCustomization::ColorPicked(FLinearColor SelectedColor)
   {
      if (MyAsset.IsValid())
      {
         MyAsset.Get()->ModColor = SelectedColor.ToFColor(false).ToHex();
      }
   }
   ```

  </details>
- <details><summary>自定义资源蓝图Pin控件 SGraphPinModAsset.h/cpp</summary>

   ```cpp
   #pragma once

   #include "CoreMinimal.h" 	
   #include "SGraphPin.h"

   /**
   * 
   */
   class DEMOZEROMOD_API SGraphPinModAsset: public SGraphPin
   {
   public:
      SLATE_BEGIN_ARGS(SGraphPinModAsset) {}
      SLATE_END_ARGS()
      void Construct(const FArguments& InArgs, UEdGraphPin* InPin);
   protected:
      virtual FSlateColor GetPinColor() const override {
         return
            FSlateColor(FColor::Black);
      };
      virtual TSharedRef<SWidget> GetDefaultValueWidget() override;
      void ColorPicked(FLinearColor SelectedColor);
   };

   // cpp
   #include "SGraphPinModAsset.h"
   #include "ModAsset.h"
   #include "Runtime/AppFramework/Public/Widgets/Colors/SColorPicker.h"
   //#include "Widgets/Colors/SColorPicker.h"

   void SGraphPinModAsset::Construct(const FArguments& InArgs, UEdGraphPin* InPin)
   {
      SGraphPin::Construct(SGraphPin::FArguments(), InPin);
   }

   TSharedRef<SWidget> SGraphPinModAsset::GetDefaultValueWidget()
   {
      return SNew(SColorPicker).OnColorCommitted(this, &SGraphPinModAsset::ColorPicked);
   }

   void SGraphPinModAsset::ColorPicked(FLinearColor SelectedColor)
   {
      UModAsset* NewValue = NewObject<UModAsset>();
      NewValue->ModColor = SelectedColor.ToFColor(false).ToHex();
      GraphPinObj->GetSchema()->TrySetDefaultObject(*GraphPinObj, NewValue);
   }

   ```

  </details>
- <details><summary>自定义资源蓝图Pin工厂 ModAssetGraphPinFactory.h/cpp</summary>

   ```cpp
   #pragma once

   #include "CoreMinimal.h"
   #include "EdGraphUtilities.h"


   /**
   * 
   */
   class DEMOZEROMOD_API FModAssetGraphPinFactory : public FGraphPanelPinFactory
   {
   public:
      virtual TSharedPtr<SGraphPin> CreatePin(UEdGraphPin* Pin) const override;
   };

   // cpp
   #include "ModAssetGraphPinFactory.h"
   #include "ModAsset.h"
   #include "SGraphPinModAsset.h"

   TSharedPtr<SGraphPin> FModAssetGraphPinFactory::CreatePin(UEdGraphPin* Pin) const
   {
      if (Pin->PinType.PinSubCategoryObject == UModAsset::StaticClass())
      {
         return SNew(SGraphPinModAsset, Pin);
      }
      else
      {
         return nullptr;
      }
   }

   ```

  </details>
- <details><summary>自定义控制台命令 DemoZeroModCommands.h</summary>

   ```cpp
   #pragma once

   //#include "CoreMinimal.h"
   #include "Framework/Commands/Commands.h"
   #include "EditorStyleSet.h"

   class FDemoZeroModCommands: public TCommands<FDemoZeroModCommands>
   {
   public:
      FDemoZeroModCommands()
         :TCommands<FDemoZeroModCommands>
         (FName(TEXT("DemoZero_Mod")),
            FText::FromString("DemoZero Mod Commands"), NAME_None,
            FEditorStyle::GetStyleSetName())
      {
      };
      virtual void RegisterCommands() override;
      TSharedPtr<FUICommandInfo> MyButton;
   };

   // cpp
   #include "DemoZeroModCommands.h"

   void FDemoZeroModCommands::RegisterCommands()
   {
   #define LOCTEXT_NAMESPACE ""
      UI_COMMAND(MyButton, "DemoZero", "Demo Zero Mod Command", EUserInterfaceActionType::Button, FInputGesture());
   #undef LOCTEXT_NAMESPACE
   }

   ```

  </details>

## 游戏运行时框架
> GamePlay Framework

### 帧

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
2. [知乎专栏：现代图形引擎入门指南](https://www.zhihu.com/column/c_1635772272538648576)
3. [Unreal Property System Reflection](https://www.unrealengine.com/zh-CN/blog/unreal-property-system-reflection)