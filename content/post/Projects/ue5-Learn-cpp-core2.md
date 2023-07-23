---
title: "UE5学习：Cpp引擎核心篇（二）"
date: 2023-07-13T14:21:07+08:00
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
本章节总结一些UE中的核心子系统的使用方式和原理
<!--more-->
## 组件相关
1. 模块```GameplayAbilities```：用于实现[Gameplay技能系统](https://docs.unrealengine.com/5.2/zh-CN/gameplay-ability-system-for-unreal-engine/)，GameplayEffects、GameplayTags、GameplayTasks等均与之联系较多。
    > 注：需添加模块AIModule、GameplayAbilities、GameplayTasks

    > 参考：[虚幻插件GAS分析 知乎专栏](https://zhuanlan.zhihu.com/p/417226776)、[[UE5]使用GamePlayTag绑定UE5增强输入](https://zhuanlan.zhihu.com/p/552662789)
    1. 主要C++ API：
        - ```UGameplayAbility```：实现技能逻辑功能
            | 成员 | 类型 | 作用 | 备注 |
            | --- | --- | --- | -- |
            | CanActivateAbility | 虚函数 | 判断角色是否可以激活某能力 | 是否具备能力（是否加点） |
            | CheckCost | 虚函数 | 计算角色当前是否可以使用某能力 | 当前状态是否能释放能力（如耗蓝） |
            | ActivateAbility | 虚函数 | 编写能力激活的核心逻辑 | 不应当直接调用 |
            | SendGameplayEvent | 虚函数 | 通知某些通用事件发生 |  |
        - ```IAbilitySystemInterface```：供Actor继承以获得技能系统接口，继承后可以获取对应的UAbilitySystemComponent实例
        - ```UAbilitySystemComponent```：Actor和技能系统交互的组件
            | 成员 | 类型 | 作用 | 备注 |
            | --- | --- | --- | -- |
            | GiveAbility | 函数 |  |  |
            | BindToInputComponent | 虚函数  | 绑定输入事件 |  |
            | InitStats | 函数 | 初始化角色Gameplay属性 |  |
            | ApplyGameplayEffectToTarget | 函数 | 对目标施加某种Effect |  |
        - ```FGameplayAbilitySpec```：存储技能信息、等级、回调等
        - ```FGameplayAbilitySpecHandle```：处理技能生效
        - ```UGameplayAbilitySet```：技能集合
            | 成员 | 类型 | 作用 | 备注 |
            | --- | --- | --- | -- |
            | GiveAbilities | 函数 |  |  |
            | Abilities | 变量 | 所有存储的技能 |  |
            > 该类型只是官方的一个示例，可以根据自己的需要管理技能
    2. 其他C++ API
        - ```UAttributeSet```：存储Actor和Gameplay相关的重要属性
        - ```UGameplayEffect```：编写对Actor的Gameplay属性有影响的逻辑内容（如各类buff、debuff），Effect对Gameplay的影响方式，主要是通过修改Attribute的值来完成的
            - ```FGameplayModifierInfo```：描述Effect生效时对Attribute施加影响的逻辑（数学运算）
                - ```EGameplayModOp```：提供基础的数学运算类型，加减乘除
            - ```FActiveGameplayEffectHandle```：用于将Effect和Actor所属的AbilitySystemComponent绑定
        - 模块```GameplayTasks```：用于包装一些游戏性功能到可重复使用对象中
            - ```UGameplayTask```：继承该类并实现一个Task的逻辑
                | 成员 | 类型 | 作用 | 备注 |
                | --- | --- | --- | -- |
                | Activate | 虚函数 | 编写激活一个Task时所需进行的工作 |  |
            - ```IGameplayTaskOwnerInterface```：父类，注意UGameplayAbility也是其子类
            - ```UGameplayTasksComponent```：Actor和Task子系统接口通信的组件
                | 成员 | 类型 | 作用 | 备注 |
                | --- | --- | --- | -- |
                | AddTaskReadyForActivation | 函数 | 添加一个可激活的Task |  |
                | NewTask | 静态函数 | 实例化一个Task | 类似NewObject，Task专用 |
            > 这里的Task，并非玩家游玩过程中的某个任务，而是指一个技能释放过程中，需要完成的各类业务逻辑。
        - 模块```GameplayTags```：根据需要，可以绑定Tag到技能、角色、物体等各种场景
    3. 示例代码
        ```cpp
        // gamemode 
        // gameplay ability绑定
        // 自定义attribute
        // 实例化gameplay effect
        ```
## 网络通信
1. 常规通信
    - UE对HTTP、UDP、WebSocket都给予了支持，下面可见代码示例
        ```cpp

        ```
2. 游戏同步
    1. 概述：UE内部实现了TCP、UDP、HTTP、Websocket等协议。但如果打算使用UE自带的Dedicated Server进行服务器端开发，需要自行从源代码编译UE。
    2. 原理：

