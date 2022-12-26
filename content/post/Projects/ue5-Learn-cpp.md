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
| 名称 | 含义 | 重要成员 |
| --- | --- | --- |
| ACharacter | 角色类 | SkeletalMesh、GetCapsuleComponent()、GetMesh() |

## 常用基本类型
| 名称 | 含义 | 注意 | 常用成员 |
| --- | --- | --- | --- |
| UStaticMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 静态网格体组件，即一个UStaticMesh的实例 | 可能被垃圾回收，需要用UPROPERTY进行标记 | SetupAttachment、SetStaticMesh |
| USkeletalMeshComponent&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 可动画化的网格体组件 | 也需用UPROPERTY标记 | |
| UStaticMesh、USkeletalMesh | 静态、骨骼网格体底层存储类型 | 需设置给特定的component才能使用 | |
| FVector、FRotator | 向量、旋转子 | 常在SetXXXRotation/Location中使用 | |


## 常用工具类/函数
| 名称 | 含义 | 注意 |
| ------ | --- | --- |
| ConstructorHelper::FObjectFinder&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | 加载某种Object资源 | 构造函数中需要给出目标地址 |
| LoadObject | 加载某种object资源 | 构造函数中给出父类Outer（可null）、目标引用名或文件地址 |
| TEXT() | 对原始代码中的文本做正确处理 | 用于静态文件路径、文字打印等多种场景，本质是添加字符串前缀L |
| CreateDefaultSuboject | 创建一个组件或者suboject | 构造函数中需要给出目标地址 |
> 注1：类型系统中，所有非Class的内容，都是一种UObject。万物皆是UObject。


## 开发要点
1. 由于C++代码带来的变化不能像蓝图一样，自动显示在编辑器中，因此需要使用**Live Coding**功能，在不重启编辑器的情况下，编译并应用C++的罪行修改。快捷键是Ctrl+Alt+F11。


## 常见功能示例
1. 添加自定义C++类：    
    1. 步骤：菜单栏→工具→新建C++类→选择合适的父类并创建
1. 


## 坑
1. C++类的更新无法在UE5编辑器内正常看到：动态编译的问题，推荐在偏好设置中，开启加载时强制编译
1. 打开已有项目，C++类在UE5编辑器内找不到了：动态编译的问题，推荐在偏好设置中，开启加载时强制编译

## 参考
1. [【虚幻5】【不适合小白观看】用C++来进行基于UE5的游戏开发（含动画蓝图）](https://www.bilibili.com/video/BV17Q4y1Y7fr)
1. [官网：C++编程 虚幻引擎编程开发的相关信息](https://docs.unrealengine.com/5.0/zh-CN/programming-with-cplusplus-in-unreal-engine/)
1. [知乎专栏：UE从点Play开始](https://zhuanlan.zhihu.com/p/512249255)
1. [Unreal Engine C++ Advanced Dark Souls Boss Fight System](https://www.youtube.com/watch?v=ANzEGECpd0g)
1. [UE5 C++ Tutorial | Introduction to Unreal Engine 5 with C++ in less than 90 Minutes](https://www.youtube.com/watch?v=nvruYLgjKkk&list=PL-m4pn2uJvXHL5rxdudkhqrSRM5gN43YN)