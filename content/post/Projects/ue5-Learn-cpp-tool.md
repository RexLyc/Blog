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
   - 在编辑器命令行内，可以使用指令关闭某个分类的输出，例如
      ```bash
      Log YourLogName off
      Log YourLogName Log
      ```
2. ```FMessageLog```
   
      <center> <br/> <img src="/images/ue/MessageLog.jpg"/> <br/> </center>

   - **编辑器**中的另一种日志，相比于UE_LOG打印到控制台的风格，```FMessageLog```偏向于事件消息，用法形如
      ```cpp
      // 定义方式
      void CreateLoggger(FName LoggerName)
      {
         // 为了进一步提供国际化能力，尽量使用LOCTEXT
         #define LOCTEXT_NAMESPACE "YourNamespace"
         #define FTEXT(x) LOCTEXT(x, x)

         FMessageLogModule& MessageLogModule =  
            FModuleManager::LoadModuleChecked<FMessageLogModule>("MessageLog");
         FMessageLogInitializationOptions InitOptions;
         InitOptions.bShowPages = true;
         InitOptions.bShowFilters = true;
         FText LogListingName = FTEXT("YourLogList");
         MessageLogModule.RegisterLogListing(LoggerName, LogListingName, InitOptions);

         // 在必要的时候取消宏定义
         #undef LOCTEXT_NAMESPACE
      }

      // 在任意位置
      MyGameMode::MyGameMode()
      {
         // 
         #define LOCTEXT_NAMESPACE "YourNamespace"
         #define FTEXT(x) LOCTEXT(x, x)

         FName LoggerName("YourLogger");
         CreateLogger(LoggerName);
         FMessageLog logger(LoggerName);
         logger.Warning(FTEXT("Message from MyGameMode"));

         // 
         #undef LOCTEXT_NAMESPACE
      }
      ```
3. ```GEngine->AddOnScreenDebugMessage```
   - 用于打印日志到窗口，可以指定位置、颜色
4. ```FString::Printf```、```FString::Format```
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
2. ```TFieldIterator<A>```
   - 用于遍历一个```UClass```
      ```cpp
      // 模板参数起到过滤作用，如果换做使用UField，则会遍历UClass中的所有元素
      for (TFieldIterator<UStruct> PropIt(GetClass()); PropIt; ++PropIt)
      {
         UStruct* Property = *PropIt;
         // Do something with the property
      }
      ```
3. ```TActorIterator<A>```
   - 用于遍历一系列以A为基类的```Actor```
      ```cpp
      for(TActorIterator<AActor> It(GetWorld(),AActor::StaticClass()); It; ++it)
      {
         // Do something...
      }
      ```
4. ```TEnumAsByte<A>```
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

5. ```Cast<A>```
   - UE提供的安全转型工具，转型失败时，会获得空指针```nullptr```
      ```cpp
      // 常见用法
      A* a = Cast<A>(b);
      if(a)
      {
         // do something
      }
      ```
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
1. 数学：
   - Leap：线性插值
   - Fmod：取余数
   - FQuat：四元数旋转
   - FRotationMatrix：旋转矩阵
## 系统
1. 时钟
   - ```FTimerHandle```：定时器句柄，用于存储已创建委托的定时器实例。用法例如：
      ```cpp
      FTimerHandle myTimer;
      GetWorld()->GetTimerManager().SetTimer(myTimer
         ,FTimerDelegate::CreateLambda([this]{
            // do something
         })
      );
      ```

## 资源工具
1. ```ConstructorHelpers::FObjectFinder```
   - 用于在**构造函数**中加载静态资源，必须写绝对路径
   - 加载结果，即资源指针可以从```FObjectFinder.Object```获取

## 字符串和国际化
1. ```FText```
   - 侧重于存储、处理在游戏运行时向屏幕渲染使用的各类文本，具有国际化能力。其使用可以搭配数个宏。
   - 另外，```FText```也提供了对于数字、百分比、日期时间等内容，优化字符串显示效果的API。
   - 参考：[Text Localization](https://docs.unrealengine.com/5.2/en-US/text-localization-in-unreal-engine/)
   ```cpp
   // 定义一个可翻译文本的命名空间
   #define LOCTEXT_NAMESPACE "MyNamespace"
   // 即使在一个命名空间内，也依然可以定义其属于其他命名空间的key-value
   FText constFTextHelloWorld= NSLOCTEXT("MyOtherNamespace","HelloWorld","Hello World!")
   // 如果不指定命名空间，则默认是当前的
   FText constFTextGoodbyeWorld= LOCTEXT("GoodbyeWorld","Goodbye World!")
   #undef LOCTEXT_NAMESPACE
   ```
2. ```FString```
   - 侧重于在内部存储、计算各类字符串，非常接近```std::string```但是提供了大量字符串API。
   - 运算符重载```operator/()```，用于路径拼接。
3. ```FName```
   - 不区分大小写，会使用hash进行快速的比较，侧重于描述名称。适合用于存储资源名称。

## 渲染
1. ```FMeshBuilder```
2. ```FVertexBuffer```
3. ```FIndexBuffer```
4. ```FPrimitiveSceneProxy```
5. ```UParticleSystem```
6. 各种Proxy

## 物理引擎
1. 碰撞检测
   1. 分类：无视碰撞、通知重叠、禁止重叠
   2. 工具类：
      1. FCollisionObjectQueryParams
      2. SweepSingleByObjectType
      3. FCollisionShape
      4. FCollisionQueryParams
2. 射线检测
   1. Hit
   2. 工具类
      1. FHitResult

## 参考
1. [Unreal Engine 5.2 API Reference](https://docs.unrealengine.com/5.2/en-US/API/)