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
1. ```UE_LOG```
   - 用于打印日志的宏，根据日志级别其内容可以打印到命令行、文件
2. ```GEngine->AddOnScreenDebugMessage```
   - 用于打印日志到窗口，可以指定位置、颜色
3. ```FString::Printf```、```FString::Format```
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
1. ```TSubclassOf<A>```
   - 用于声明一个类型应当是A的子类型，多用于支持C++和蓝图的互操作
      ```cpp
      // 用于接收任意从UObject派生的类类型
      UPROPERTY(EditAnywhere, BlueprintReadWrite)
      TSubclassOf<UObject> UClassOfAny;
      ```
   - 实践：当C++中计划使用蓝图时，可以先定义一个C++类A，并派生蓝图B。将B放置于任何UClassOfAny的出现位置
1. ```TFieldIterator<A>```
   - 用于遍历一个```UClass```
      ```cpp
      // 模板参数起到过滤作用，如果换做使用UField，则会遍历UClass中的所有元素
      for (TFieldIterator<UStruct> PropIt(GetClass()); PropIt; ++PropIt)
      {
         UStruct* Property = *PropIt;
         // Do something with the property
      }
      ```
1. ```TEnumAsByte<A>```
   - 在使用C++98版风格的枚举值定义时，用于将枚举值作为类型。
      ```cpp
      UENUM(BlueprintType)
      namespace ETest98
      {
         enum Type
         {
            Test1,
            Test2,
            Test3,
            Test4,
         };
      }

      UCLASS()
      class TEST_API UMyObject : public UObject
      {
         GENERATED_BODY()

      public:
         UPROPERTY(BlueprintReadWrite,EditAnywhere)
         // 编译错误，不能作为类型定义变量
         // ETest98::Type Type;
         TEnumAsByte<ETest98::Type> Type;
      };
      ```
   - 应当只在C++11版枚举值无法满足需求时（超过uint8的数量）使用。
   > 参考：[UE4枚举类型使用总结](https://zhuanlan.zhihu.com/p/492630586)

## 容器
1. ```TArray```
   - 动态数组（vector）
2. ```TMap```
   - 字典
3. ```TSet```
   - 集合

## 智能指针
1. 概述：
   - 智能指针默认都是**线程不安全**的，可在创建时添加模板参数```ESPMode::ThreadSafe```
2. ```TSharedRef``` & ```TSharedPtr```
   - 共享指针、共享引用（不许为空）
3. ```TWeakPtr```
   - 弱引用共享指针
4. ```TUniquePtr```
   - unique指针
5. 工具类和函数
   1. ```TSharedFromThis```：继承该类以继承一些便于共享的成员函数
   2. ```MakeShared``` & ```MakeShareable```：从裸指针创建共享指针的函数
   3. ```StaticCastSharedRef``` & ```StaticCastSharedPtr```：用于进行下转型的函数
   4. ```ConstCastSharedRef``` & ```ConstCastSharedPtr```：将const转为mutable
> 参考：[官方文档：UE5.2 Unreal Smart Pointer Library](https://docs.unrealengine.com/5.2/en-US/smart-pointers-in-unreal-engine/)

## 算法 & 数据结构
1. 插值：
   - Leap线性插值
   - 

## 资源工具
1. ```ConstructorHelpers::FObjectFinder```
   - 用于在**构造函数**中加载静态资源，必须写绝对路径
   - 加载结果，即资源指针可以从```FObjectFinder.Object```获取

## 渲染
1. ```FMeshBuilder```
2. ```FVertexBuffer```
3. ```FIndexBuffer```
4. ```FPrimitiveSceneProxy```
5. ```UParticleSystem```
6. 各种Proxy

## 物理引擎
