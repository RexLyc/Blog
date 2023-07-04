---
title: "UE5学习：UI篇"
date: 2023-06-13T23:01:54+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
编写好看好用的UI是用户能够获得体验的重点之一，本文记录UE中UI开发相关的内容。
<!--more-->
## HUD和SlateUI
1. HUD类型是UE中对于界面进行控制的主要类型。该类型实例存储于```GameMode```中，自定义HUD也需要在其中创建实例。
2. ```Canvas```
   - 是从UE3流传下来的简单HUD实现，仍然可以使用，但相对功能没有那么强大。
   - 示例代码（在HUD类型中）
        ```cpp
        class MYPROJECT_API MyHUD: public AHUD
        {
        public:
            virtual void DrawHUD() override
            {
                Super::DrawHUD();
                // 绘制文字
                Canvas->DrawText(GEngine->GetSmallFont(), TEXT("hello"),10,10);
                // 创建一个进度条
                FCanvasBoxItem ProgressBar(FVector2D(5,25),FVector2D(100,5));
                // 绘制进度条
                Canvas->DrawItem(ProgressBar);
                // DrawRect是AHUD的函数
                DrawRect(FLinearColor::Blue,5,25,100,5);
            }
        }
        ```
3. Slate核心原理：
   1. ```FSlateApplication```：UI调度单例
   2. ```SWindow```：一个SlateUI的顶层窗口
   3. ```SWidget```：控件
      1. ```SPanel```：子类都是具有槽（Slot）的，可以添加子控件
4. 开发流程：
    - 编辑器中新建C++类型，继承HUD类型：实现自己的HUD类。HUD类是UI的入口，在这个类型中创建具体的SlateUI实例，并在适当的操作后删除SlateUI实例返回游戏界面
    - 编辑器中新建C++类型（SlateUI），继承SCompoundWidget：实现自己的窗口，窗口是UI的具体绘制内容，在这个类型中，需要设计UI资源加载，并设计UI布局，UI控件，编写UI操作回调
    - SlateUI，即SCompoundWidget为代表的一系列类型系统，使用一种相对特殊的C++语法，相关内容可以参考官网链接
    > 注1：SlateUI类型在当前版本（5.2）中，不在UE5引擎的垃圾回收机制内，因此需要自行管理，推荐使用智能指针的方式

    > 注2：SlateUI类型，在定义、实现过程中，有大量不符合常规思路的代码、宏，需要加以理解
5. 常用SlateUI：
    - 控件：SOverlay、STextBlock、SCanvas、SVerticalBox、SEditableText、SButton
    - 函数：HAlign、VAlign、Padding、Text、Font、ColorAndOpacity、Size、Position、OnXXXX
6. 类型、函数、宏
    | 名称 | 类型 | 含义 | 注意 |
    | --- | --- | --- | --- |
    | SLATE_BEGIN_ARGS | 宏 | Slate固定范式 | |
    | SLATE_ARGUMENT | 宏 | Slate固定范式 | 输入一个类型，一个变量，提供到Construct的InArgs内 |
    | SLATE_END_ARGS| 宏 | Slate固定范式 | |
    | BEGIN_SLATE_FUNCTION_BUILD_OPTIMIZATION | 宏 | Slate实现部分固定范式 | 将所有SlateUI窗口类的实现部分包含在内 |
    | END_SLATE_FUNCTION_BUILD_OPTIMIZATION | 宏 | Slate实现部分固定范式 | 将所有SlateUI窗口类的实现部分包含在内 |
    | Construct | SlateUI窗口类成员函数 | UI业务的真正构造函数 | 其参数内就包含这SLATE_ARGUMENT传递的变量 |
    | SCanvas | SlateUI控件类类型 | 一般用于定义一个UI区域，其内完成一个独立功能 | 一般使用TSharedRef\<\>来共享 |
    | ChildSlot | SlateUI窗口类成员变量 | UI描述起点 | 该类型已对operator[]进行重载 |
    | SNew | 宏 | 实例化一个UI控件、窗口 | 在ChildSlot下、HUD类内广泛使用 |
    | SAssignNew | 宏 | 实例化一个UI控件、窗口，并赋值给参数 | 在HUD类内广泛使用 |
    | XX::Slot() | SlateUI控件类成员函数 | 用于创建子槽，以定义子控件 |
    | GEngine->GameViewport | GEngine成员 | 获取视口 | 用于获取视口，后续绑定、删除UI |
    | (Add/Remove)ViewportWidgetContent | ViewPort成员函数 | 创建、删除UI窗口 | 内部一般使用智能指针 |
    | XXXXGameModeBase::HUDClass | 游戏模式成员变量 | 用于将HUD类型绑定给游戏模式 | 和玩家控制器、默认玩家一样，需要进行复制 |
    | FSlateBrush | 笔刷类型 | 用于画图 | |
    | FReply | 反馈类型 | 用于各类OnXXX绑定的回调函数的返回类型 | 回调最后必须调用handled以标记结束 |
    | EVisibility | 枚举值 | 用于描述Slate对象可见性 |  |
7. 代码示例
    - 自定义窗口.h/.cpp
        <details>
        <summary>代码示例</summary>

        ```cpp
        #pragma once

        #include "CoreMinimal.h"
        #include "MyHUD.h"
        #include <Runtime/Slate/Public/Widgets/SCanvas.h>
        #include <Runtime/Slate/Public/Widgets/Input/SEditableText.h>
        #include <Runtime/Slate/Public/Widgets/SWeakWidget.h>
        #include "Widgets/SCompoundWidget.h"
        class CPPLEARN_API SMyCompoundWidget : public SCompoundWidget
        {
        public:
            SLATE_BEGIN_ARGS(SMyCompoundWidget)
            {}
            SLATE_ARGUMENT(TWeakObjectPtr<class AMyHUD>, myHUD)
            SLATE_END_ARGS()

            /** Constructs this widget with InArgs */
            void Construct(const FArguments& InArgs);

            TWeakObjectPtr<AMyHUD> myHUD;

            TSharedRef<SCanvas> LoginPanel();
            TSharedRef<SCanvas> Passworld();
            TSharedRef<SCanvas> Button();

            FSlateBrush brush;

            void PassworldChanged(const FText& text);

            FReply PassWorldClick();
        };

        // ================================================= //
        #include "SMyCompoundWidget.h"
        #include "SlateOptMacros.h"

        BEGIN_SLATE_FUNCTION_BUILD_OPTIMIZATION
        void SMyCompoundWidget::Construct(const FArguments& InArgs)
        {
            /*
            ChildSlot
            [
                // Populate the widget
            ];
            */
            myHUD = InArgs._myHUD;

            const FString path = FPaths::ProjectContentDir() + "MaoKenShiJinHei.ttf";
            FSlateFontInfo robot(path, 30);
            robot.LetterSpacing = 100;


            brush.SetResourceObject(LoadObject<UTexture2D>(nullptr, TEXT("Texture2D'/Game/Poor.Poor'")));
            

            ChildSlot
            [
                SNew(SOverlay)
                +SOverlay::Slot()
                    .HAlign(HAlign_Center)
                    .VAlign(VAlign_Top)
                    .Padding(0,10,0,0)
                    [
                        SNew(STextBlock)
                        .ColorAndOpacity(FLinearColor::Black)
                        .ShadowColorAndOpacity(FLinearColor::Black)
                        .ShadowOffset(FIntPoint(-1,0))
                        .Font(robot)
                        .Text(FText::FromString(TEXT("界面测试")))
                    ]
                +SOverlay::Slot()
                    .HAlign(HAlign_Center)
                    .VAlign(VAlign_Center)
                    .Padding(10,0,0,55)
                    [
                        LoginPanel()
                    ]
            ];
        }

        TSharedRef<SCanvas> SMyCompoundWidget::LoginPanel() {
            return SNew(SCanvas)
                + SCanvas::Slot()
                        .HAlign(HAlign_Center)
                        .VAlign(VAlign_Center)
                        .Size(FVector2D(550, 280))
                        [
                            SNew(SBorder).HAlign(HAlign_Fill).VAlign(VAlign_Fill).BorderImage(&brush)
                            [
                                SNew(SVerticalBox)
                                + SVerticalBox::Slot().Padding(10, 20, 0, 10)
                                    [
                                        Passworld()
                                    ]
                                + SVerticalBox::Slot().Padding(10, 20, 0, 10)
                                    [
                                    Button()
                                    ]
                            ]
                        ];
        }

        TSharedRef<SCanvas> SMyCompoundWidget::Passworld() {
            return SNew(SCanvas) + SCanvas::Slot().Position(FVector2D(0, 0)).Size(FVector2D(500, 80))
                [
                    SNew(SEditableText)
                    .OnTextChanged(this,&SMyCompoundWidget::PassworldChanged)
                    .HintText(FText::FromString("TEXT"))
                    .ColorAndOpacity(FLinearColor::White)
                    .IsPassword(true)
                ];
        }
        TSharedRef<SCanvas> SMyCompoundWidget::Button() {
            return SNew(SCanvas) 
                + SCanvas::Slot().Position(FVector2D(0,0)).Size(FVector2D(500, 80))
                        [
                            SNew(SButton)
                            .HAlign(HAlign_Center)
                            .VAlign(VAlign_Center)
                            .Text(FText::FromString("Press Me"))
                            .OnClicked(this,&SMyCompoundWidget::PassWorldClick)
                        ];
        }

        void SMyCompoundWidget::PassworldChanged(const FText& text) {
            UE_LOG(LogTemp, Log, TEXT("%s"), *text.ToString());
        }


        FReply SMyCompoundWidget::PassWorldClick() {
            UE_LOG(LogTemp, Log, TEXT("pressed!"));
            if (myHUD.IsValid()) {
                if (APlayerController* pc = myHUD->PlayerOwner) {
                    pc->ConsoleCommand("quit");
                }
            }
            return FReply::Handled();
        }
        END_SLATE_FUNCTION_BUILD_OPTIMIZATION
        ```
        </details>
    - 自定义HUD的.h/.cpp
        <details>
        <summary>代码示例</summary>

        ```cpp
        #include "CoreMinimal.h"
        #include "GameFramework/HUD.h"
        #include "SMyCompoundWidget.h"
        #include "MyHUD.generated.h"

        UCLASS()
        class CPPLEARN_API AMyHUD : public AHUD
        {
            GENERATED_BODY()
            
        protected:
            virtual void BeginPlay() override;
        public:

            TSharedPtr<class SMyCompoundWidget> myUI;
            TSharedPtr<class SWidget> myUIContainter;

            void OpenMenu();

            void CloseMenu();
        };

        // ================================================= //
        #include "MyHUD.h"

        void AMyHUD::BeginPlay() {
            Super::BeginPlay();
            UE_LOG(LogTemp, Log, TEXT("BeginPlay Setup UI"));
            OpenMenu();
        }

        void AMyHUD::OpenMenu()
        {
            if (GEngine && GEngine->GameViewport) {
                myUI = SNew(SMyCompoundWidget).myHUD(this);

                GEngine->GameViewport->AddViewportWidgetContent(
                    SAssignNew(myUIContainter, SWeakWidget).PossiblyNullContent(myUI.ToSharedRef())
                );

                myUI->SetVisibility(EVisibility::Visible);
                if (PlayerOwner) {
                    PlayerOwner->bShowMouseCursor = true;
                    // 设置将输入仅挂载到UI
                    //PlayerOwner->SetInputMode(FInputModeUIOnly());
                }
            }
        }

        void AMyHUD::CloseMenu()
        {
            UE_LOG(LogTemp, Log, TEXT("Close Menu ing"));
            if (GEngine && GEngine->GameViewport && myUIContainter.IsValid()) {
                // 这里选择直接删除
                GEngine->GameViewport->RemoveViewportWidgetContent(myUIContainter.ToSharedRef());

                if (PlayerOwner) {
                    PlayerOwner->bShowMouseCursor = false;
                    // 设置将输入仅挂载到游戏
                    //PlayerOwner->SetInputMode(FInputModeGameOnly());
                }
            }
        }

        ```
        </details>
## UMG
1. UMG是基于SlateUI开发的一套UI框架，相对来说更容易使用
1. 使用流程：
    1. 在Build.cs中添加UMG模块
    1. 创建以UserWidget为父类的自定义子类A
    1. 创建继承了子类A的控件蓝图（用户界面->控件蓝图）
    1. 必要的时候将子类A的成员变量和蓝图中的控件进行过绑定
    1. 在蓝图中编辑界面，或在C++代码中编辑界面
    1. 在BeginPlay中LoadClass、CreateWidget
    > 注1：蓝图编辑时，需要先添加一个画布面板，才能添加各类具体的UI控件
1. 相关类型、函数、宏
| 名称 | 类型 | 含义 | 注意 |
| --- | --- | --- | --- |
| UUserWidget | UMG基类 | 提供UMG框架的基础功能 | 继承该类来实现自己的UI |
| UPROPERTY(meta=(BindWidget)) | 宏 | 惯用写法，将C++成员变量和蓝图控件绑定 | 必须完全同名 |
| UButton | 按钮类型 | 按钮 | 各种控件的类型可以在蓝图中查看（右上角） |
| FInputModeUIOnly | 类 | 输入模式参数 | 设置输入模式的各类可配置参数（如仅UI，仅游戏） |
| AddToViewport() | 函数 | 将当前UI实例添加到视口 | CreateWidget后调用 |
| RemoveFromParent() | 函数 | 将当前UI实例从托管的父类中移除 | 一般搭配对控制输入模式的恢复 |
| IUserObjectListEntry | 类 | 用于自定义列表单元项 | 自定义时需要继承该类 |
| NativeXXXXX | 继承函数 | 各类UI控件类型内存在的回调函数 | 在自定义的子类型中重写该类型的函数 |

1. 示例代码
    - 自定义列表单元项.h/.cpp
    ```cpp
    // MyListEntry.h
    #include "CoreMinimal.h"
    #include "Blueprint/UserWidget.h"
    #include "Blueprint/IUserObjectListEntry.h"
    #include <Runtime/UMG/Public/Components/Button.h>
    #include <Runtime/UMG/Public/Components/TextBlock.h>
    #include "MyListEntry.generated.h"

    /**
    * 
    */
    UCLASS()
    class CPPLEARN_API UMyListEntry : public UUserWidget, public IUserObjectListEntry
    {
        GENERATED_BODY()
    public:
        virtual void NativeOnListItemObjectSet(UObject* item);

        virtual void NativeOnItemSelectionChanged(bool isSelected);

        UPROPERTY(meta = (BindWidget))
        UButton* Button_0;

        UPROPERTY(meta = (BindWidget))
        UTextBlock* Text_0;
    };


    // MyListEntry.cpp
        
    #include "MyListEntry.h"
    #include "MyListEntryContent.h"

    void UMyListEntry::NativeOnListItemObjectSet(UObject* item) {

        const FString path = FPaths::ProjectContentDir() + "MaoKenShiJinHei.ttf";
        FSlateFontInfo robot(path, 30);
        robot.LetterSpacing = 100;
        UMyListEntryContent* content = Cast<UMyListEntryContent>(item);
        Text_0->SetText(FText::FromString(content->title));
        Text_0->SetFont(robot);
    }

    void UMyListEntry::NativeOnItemSelectionChanged(bool isSelected) {

    }

    ```
    - 自定义UI窗口.h/.cpp
    ```cpp
    // MyUserWidget.h
    #pragma once

    #include "CoreMinimal.h"
    #include "Blueprint/UserWidget.h"
    #include <Runtime/UMG/Public/Components/Button.h>
    #include <Runtime/UMG/Public/Components/ListView.h>
    #include "MyListEntryContent.h"
    #include "HTTPRequest.h"
    #include "MyUserWidget.generated.h"

    /**
    * 
    */
    UCLASS()
    class CPPLEARN_API UMyUserWidget : public UUserWidget
    {
        GENERATED_BODY()

    public:

        // 以相同的名字和BindWidget来和蓝图双向绑定
        UPROPERTY(meta=(BindWidget))
        UButton* Button_0;

        void Open();

        UPROPERTY(meta = (BindWidget))
        UListView* ListView_0;
        
        // 为了使用内部的事件
        UPROPERTY(EditInstanceOnly, Category = "Basic Config")
        UHTTPRequest* httpRequest;
       	
        UFUNCTION()
        void ShowLogin(FString result);
    };

    // MyUserWidget.cpp
    #include "MyUserWidget.h"
    void UMyUserWidget::Open()
    {
        FSlateBrush brush;
        brush.SetResourceObject(LoadObject<UTexture2D>(nullptr, TEXT("Texture2D'/Game/Poor.Poor'")));
        FButtonStyle style;
        style.SetNormal(brush);

        Button_0->SetStyle(style);

        AddToViewport();
        SetVisibility(ESlateVisibility::Visible);
        bIsFocusable = true;
        APlayerController* playerController = GetWorld()->GetFirstPlayerController();
        if (!playerController) {
            return;
        }
        FInputModeUIOnly uiOnly;
        uiOnly.SetWidgetToFocus(TakeWidget());
        uiOnly.SetLockMouseToViewportBehavior(EMouseLockMode::DoNotLock);
        playerController->SetInputMode(uiOnly);
        playerController->SetShowMouseCursor(true);

        httpRequest = NewObject<UHTTPRequest>();

        TArray<FString> itemNames;
        itemNames.Emplace("ABC");
        // 自定义的一个继承UObject的专用列表单元数据类型
        TArray<UMyListEntryContent*> list;
        for (FString item : itemNames) {
            UMyListEntryContent* temp = NewObject<UMyListEntryContent>();
            temp->title = item;
            list.Emplace(temp);
        }
        ListView_0->SetListItems(list);

        // loginResult是由DECLARE_MULTICAST_DELEGATE_OneParam(FLoginResult,FString)定义
        httpRequest->loginResult.AddUObject(this, &UMyUserWidget::ShowLogin);
        // 网络通信示例，其对应注册的Receive中应当调用loginResult.BroadCast(...)
        httpRequest->Send("http://127.0.0.1:8080/weblab/remote/echo", "hello world");
    }

    void UMyUserWidget::ShowLogin(FString result) 
    {
        UE_LOG(LogTemp, Log, TEXT("Login : %s"),*result);
    }
    ```
    - 主角色，或GameMode子类的BeginPlay：
    ```cpp
    void XXXX:BeginPlay() {
        // UMG UI, 必须添加_C后缀
        FString path = "/Game/UI/Your_WBP.Your_WBP_C";
        UClass* widget = LoadClass<UUserWidget>(nullptr, *path);
        UMyUserWidget* myWidget = CreateWidget<UMyUserWidget>(GetWorld(), widget);
        if(myWidget!=nullptr)
            login->Open();
        else
            UE_LOG(LogTemp, Log, TEXT("widget create failed"));
    }
    ```

## 编辑器和项目设置
1. 在项目设置中，可以配置UI的缩放倍率曲线（DPI Curve，缩放-分辨率），从而达到在不同分辨率下的最优效果。
   - 尚未解决：如何在代码中动态的修改这部分？

## 参考
1. [现代图形引擎入门指南（Unreal Engine）— Slate开发 - Italink的文章 - 知乎](https://zhuanlan.zhihu.com/p/636153935)
2. [官方UE5.2 Slate架构](https://docs.unrealengine.com/5.2/zh-CN/understanding-the-slate-ui-architecture-in-unreal-engine/)