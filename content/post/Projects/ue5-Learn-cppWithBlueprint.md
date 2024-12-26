---
title: "UE5学习：Cpp和蓝图合作篇"
date: 2023-01-03T21:51:52+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
draft: true
---
UE5最强大的使用方式应该是将蓝图和C++混合使用，充分利用两种开发模式的强处，强强联手，本章节将给出一些实际的应用案例。
<!--more-->
## 动画蓝图和混合空间
1. 问题：用纯C++做带有衔接的动画相对比较困难，而蓝图能很好的弥补这一点
1. 蓝图相关术语：
    - 动画蓝图：创建、修改动画的专用蓝图，该蓝图实际上就是UAnimInstance类的一种子类。开发者可以根据情况，先编写一个UAnimInstance，再创建一个以该类型为子类的动画蓝图。
    <center><img src="/images/ue/CppVariableToBlueprint.jpg" width="256" height="256" ><B>从C++源文件中获取Direction等变量到蓝图中</B></center>

    <center><img src="/images/ue/BlueprintStateMachine.jpg" width="180" height="180" ><B>静止、跑步、起跳、下落、落地动作状态机设计</B></center>

    - 混合空间：允许基于输入参数，在多种动画之间进行混合
        - 水平坐标：控制水平面上横向的移动。一般的，该值是通过速度和位移方向共同计算，可以理解这个值代表了人物在水平方向发生转向。一般范围[-180,180]
        - 垂直坐标：控制水平面上纵向的移动。一般的，可以通过垂直坐标来区分起跑步和行走。即这个值代表了任务在向前、后移动
        - 最大最小轴值：字面意思，对坐标的限制
        - 平滑时间：动画参数变更时的平滑时间，0意味着禁用平滑。根据需要设置。
1. C++相关开发：
    - 编写继承UAnimInstance的子类，计算必要的动画控制字段，获取所需要控制的角色类实例。该类型的类属性、成员属性一般都需要设置为支持蓝图操作，如对类用Bluprintable、BluprintType，对成员用EditAnywhere、BluprintReadWrite。
    - 在角色类的BeginPlay位置，设置AnimInstanceClass，将动画类赋予给角色类实例。
    - 详见[UE5学习：Cpp篇]({{<ref "/content/post/Projects/ue5-Learn-cpp.md">}})
1. 注意要点：
    - 使用UAnimInstance对角色进行动画设置，和使用SetAnimation()方法进行设置是互斥的，只有最后使用的方式会生效。

## 参考
1. [【虚幻5】【不适合小白观看】用C++来进行基于UE5的游戏开发（含动画蓝图）](https://www.bilibili.com/video/BV17Q4y1Y7fr?p=15)