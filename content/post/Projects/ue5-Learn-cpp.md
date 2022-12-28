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
## 类型系统概述
1. Unreal Engine大幅拓展了C++的能力，并形成了自己的类型系统，添加C++类型时，需要遵照相应的框架规则。
1. 宏构成了Unreal Engine对C++扩展的一大部分，从类型生成到类成员的生成，都有宏的参与


## 常用宏
| 名称 | 含义 | 参数 |
| ----- | --- | --- |
| UPROPERTY | 类成员属性设置 | (UP::XXXenum，Categor="在编辑器-细节面板中的名字") |

## 核心基类
1. ACharacter：角色类型通用的基类
| 成员名称 | 成员类型 | 含义 | 
| --- | --- | --- |
| GetCapsuleComponent() | 继承函数 | 获取碰撞胶囊组件 |
| GetMesh() | 继承函数 |获取网格体，用于进一步设置SkeletalMesh等 |
| bUseControllerRotationXXXX | 继承变量 | 指示变量：是否使用控制器的输入控制旋转 |
| SetupPlayerInputComponent | 重写函数 | 用于将输入事件绑定到用户输入组件 |
| AutoPossessPlayer | 继承变量 | 用于记录该角色相机视角是否为初始视角（Player0） |
| AddMovementInput | 继承函数 | 用于提供橘色的移动方向和移动量 |

1. AController：角色控制通用的基类
| 成员名称 | 成员类型 | 含义 | 
| --- | --- | --- |
| InputComponent | 继承变量 | 默认为空的输入组件 |

1. AGameModeBase：游戏模式基类
| 成员名称 | 成员类型 | 含义 |
| --- | --- | --- |
| PlayerControllerClass | 继承变量 | 默认玩家控制器 |
| DefaultPawnClass | 继承变量 | 默认角色 |

## 广泛继承的函数
| 名称 | 继承来源 | 含义 |
| --- | --- | --- |
| StaticClass() | | 创建并返回所属类型的静态实例 |

## 常用基本类型
| 名称 | 含义 | 注意 | 常用成员 |
| --- | --- | --- | --- |
| UStaticMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 静态网格体组件，即一个UStaticMesh的实例 | 可能被垃圾回收，需要用UPROPERTY进行标记 | SetupAttachment、SetStaticMesh |
| USkeletalMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 可动画化的网格体组件 | 也需用UPROPERTY标记 | |
| UStaticMesh、USkeletalMesh | 静态、骨骼网格体底层存储类型 | 需设置给特定的component才能使用 | |
| FVector、FRotator | 向量、旋转子 | 常在SetXXXRotation/Location中使用 | |
| USpringArmComponent | 摇臂类，常用于辅助第三人称相机 | 一般绑定到角色 | |
| UCameraComponent | 相机类型 | 常绑定到摇臂 | |
| UInputComponent | 输入组件，将输入事件绑定到具体函数 | | BindAxis、BindAction | 


## 常用工具类/函数
| 名称 | 含义 | 注意 |
| ------ | --- | --- |
| ConstructorHelper::FObjectFinder()&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 加载某种Object资源 | 构造函数中需要给出目标名称 |
| LoadObject<>() | 加载某种object资源 | 构造函数中给出父类Outer（可null）、目标引用名或文件地址 |
| TEXT() | 对原始代码中的文本做正确处理 | 用于静态文件路径、文字打印等多种场景，本质是添加字符串前缀L |
| CreateDefaultSuboject<>() | 创建一个组件或者suboject | 构造函数中需要给出目标名称 |
| FRotationMatrix() | 创建旋转矩阵 | 构造函数中给出旋转角向量 |
| RotationMatrix::GetUnitAxis() | 获取旋转矩阵对单位轴向量旋转后的结果 | 参数是EAxis::X/Y/Z |
| Cast<>() | UE5类型体系下的转型 | |
| UE_LOG() | 日志宏 | 需要提供日志类型enum，日志级别enum |
> 注1：类型系统中，所有非Class的内容，都是一种UObject。万物皆是UObject。


## 开发要点
1. 由于C++代码带来的变化不能像蓝图一样，自动显示在编辑器中，因此需要使用**Live Coding**功能，在不重启编辑器的情况下，编译并应用C++的罪行修改。快捷键是Ctrl+Alt+F11。


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
    - **尚未解决**
1. UE体系内C++类型修改后，Live Coding后，运行仍然未更新：删除原对象，重新拖动对象到关卡内。
    - **是否有自动的办法**

## 参考
1. [【虚幻5】【不适合小白观看】用C++来进行基于UE5的游戏开发（含动画蓝图）](https://www.bilibili.com/video/BV17Q4y1Y7fr)
1. [官网：C++编程 虚幻引擎编程开发的相关信息](https://docs.unrealengine.com/5.0/zh-CN/programming-with-cplusplus-in-unreal-engine/)
1. [知乎专栏：UE从点Play开始](https://zhuanlan.zhihu.com/p/512249255)
1. [Unreal Engine C++ Advanced Dark Souls Boss Fight System](https://www.youtube.com/watch?v=ANzEGECpd0g)
1. [UE5 C++ Tutorial | Introduction to Unreal Engine 5 with C++ in less than 90 Minutes](https://www.youtube.com/watch?v=nvruYLgjKkk&list=PL-m4pn2uJvXHL5rxdudkhqrSRM5gN43YN)