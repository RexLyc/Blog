---
title: "UE5学习：Cpp工具篇"
date: 2023-06-12T14:10:19+08:00
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
本篇详细总结了UE在C++中提供的各类工具性质的类、函数。
<!--more-->
## 日志和格式化
1. UE_LOG
   - 用于打印日志的宏，根据日志级别其内容可以打印到命令行、文件
2. GEngine$\to$AddOnScreenDebugMessage
   - 用于打印日志到窗口，可以指定位置、颜色
3. FString::Printf、FString::Format
   - 创建格式化字符串，后者可以自动格式化，无需填写格式字符，例如
    ```cpp
    TArray< FStringFormatArg > args;
    args.Add( FStringFormatArg( name ) );
    args.Add( FStringFormatArg( mana ) );
    FString string = FString::Format( TEXT( "Name = {0} Mana = {1}" ), args );
    UE_LOG( LogTemp, Warning, TEXT( "Your string: %s" ), *string );
    ```
> 详见：[关于日志的社区手册](https://unrealcommunity.wiki/logging-lgpidy6i)

## 泛型
1. TSubclassOf\<A\>
   - 用于声明一个类型应当是A的子类型，多用于支持C++和蓝图的互操作
      ```cpp
      // 用于接收任意UObject子类型的实例
      UPROPERTY(EditAnywhere, BlueprintReadWrite)
      TSubclassOf<UObject> UClassOfAny;
      ```
   - 实践：当C++中计划使用蓝图时，可以先定义一个C++类A，并派生蓝图B。将B放置于任何UClassOfAny的出现位置