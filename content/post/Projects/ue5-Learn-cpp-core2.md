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
## 本章宏总结
| 宏名称 | 宏作用 | 所在模块 | 注意 |
| --- | --- | --- | --- |
| GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) | 添加GAS属性getter函数 | GAS | 和VALUE_GETTER区别是返回值 |
| GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) | 添加GAS属性值getter函数 | GAS |  |
| GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) | 添加GAS属性值setter函数 | GAS |  |
| GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName) | 添加GAS属性值初始化函数 | GAS |  |

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
            ![将标签表格添加到项目设置](/images/ue/AbilityTagInSetting.jpg)

            ![创建GameplayTagTableRow表格](/images/ue/GameplayTagTableRow.jpg)
          - 添加属性有多种方法，可以在项目设置中直接添加，也可以自定义一个继承了```GameplayTagTableRow```的数据表格（DataTable），并在项目设置中进行加载。
          - ```IAbilitySystemInterface```，其实是```IGameplayTagAssetInterface```的派生接口，因此```AbilitySystemComponent```就包含了标签管理功能。如果只需要管理标签，可以只继承后者。
          - 坑：
            1. ```GameplayTag```的使用一般都依附于```GameplayEffect```，即产生一个效果时会附带标签。如果想要单独添加Tag，需要使用```AddLooseGameplayTags```，形如
                ```cpp
                FGameplayTag tag = FGameplayTag::RequestGameplayTag(FName("Attack.Base.Main"));
                AbilitySystemComponent->AddLooseGameplayTag(tag);
                ```
          - 参考：[Youtube:Gameplay Tags in Unreal Engine 5](https://www.youtube.com/watch?v=-hcAvuY3tMY)
            > 注意区分GAS系统所使用的标签和UE本身的标签系统
    3. 示例代码
        创建自定义Ability
        ```cpp
        #pragma once

        #include "CoreMinimal.h"
        #include "Abilities/GameplayAbility.h"
        #include "GameplayBaseAttack.generated.h"

        /**
        * 
        */
        UCLASS()
        class DEMOZERO_API UGameplayBaseAttack : public UGameplayAbility
        {
            GENERATED_BODY()
        public:
            // 参考UGameplayAbility虚函数，实现必要的部分

            // 技能是否已具备
            virtual bool CanActivateAbility(const FGameplayAbilitySpecHandle Handle
                , const FGameplayAbilityActorInfo* ActorInfo
                , const FGameplayTagContainer* SourceTags = nullptr
                , const FGameplayTagContainer* TargetTags = nullptr
                , OUT FGameplayTagContainer* OptionalRelevantTags = nullptr) const override;

            // 技能是否可释放
            virtual bool CheckCost(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, OUT FGameplayTagContainer* OptionalRelevantTags = nullptr) const override;

            // 激活技能
            virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;
            
            // 收到技能绑定输入事件
            virtual void InputPressed(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) override;
        };
        ```

        创建自定义Attribute
        ```cpp
        #pragma once

        #include "CoreMinimal.h"
        #include "AttributeSet.h"
        #include "AbilitySystemComponent.h"
        #include "CharacterBaseAttributeSet.generated.h"

        #define ATTRIBUTE_ACCESSOR(ClassName, PropertyName)		\
            GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
            GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName)	\
            GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName)	\
            GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

        /**
        * 
        */
        UCLASS()
        class DEMOZERO_API UCharacterBaseAttributeSet : public UAttributeSet
        {
            GENERATED_BODY()
        public:

            UCharacterBaseAttributeSet();

            UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "CharacterBaseAttribute")
            FGameplayAttributeData Hp;
            ATTRIBUTE_ACCESSOR(UCharacterBaseAttributeSet, Hp)

            UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "CharacterBaseAttribute")
            FGameplayAttributeData Mana;
            ATTRIBUTE_ACCESSOR(UCharacterBaseAttributeSet, Mana)

            UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "CharacterBaseAttribute")
            FGameplayAttributeData Scale;
            ATTRIBUTE_ACCESSOR(UCharacterBaseAttributeSet, Scale)
        };

        #undef ATTRIBUTE_ACCESSOR
        ```

        创建自定义GameEffect效果（更推荐在蓝图中使用），这里简化了，使用默认的UGameplayEffect演示
        ```cpp

        UGameplayEffect* effect = NewObject<UGameplayEffect>();
        FGameplayModifierInfo info;
        // 加法
        info.ModifierOp = EGameplayModOp::Additive;
        // 获取上述的Hp属性
        info.Attribute = UCharacterBaseAttributeSet::GetHpAttribute();
        // 每次修改的量
        info.ModifierMagnitude = FGameplayEffectModifierMagnitude(50.0f);
        // 修改器可以添加多种效果
        effect->Modifiers.Add(info);
        // 持续策略（瞬间、持续、永久）
        effect->DurationPolicy = EGameplayEffectDurationType::Infinite;
        // 持续事件
        effect->DurationMagnitude = FScalableFloat(10.0f);
        // 作用概率
        effect->ChanceToApplyToTarget = 1.0f;
        // 触发时间间隔
        effect->Period = 0.5f;
        FActiveGameplayEffectHandle handle = AbilitySystemComponent->ApplyGameplayEffectToTarget(effect, AbilitySystemComponent, 1.0f);
        ```

        创建自定义GameplayTask（这里演示了一个在一系列位置放置数字的任务）
        ```cpp
        #pragma once

        #include "CoreMinimal.h"
        #include "GameplayTask.h"
        #include "Text3DActor.h"
        #include "RoutineGameplayTask.generated.h"

        /**
        * 
        */
        UCLASS()
        class DEMOZERO_API URoutineGameplayTask : public UGameplayTask
        {
            GENERATED_BODY()
        public:
            static URoutineGameplayTask* ConstructTask(TScriptInterface<IGameplayTaskOwnerInterface> TaskOwner, TArray<FVector> Locations);

            UPROPERTY()
            TArray<FVector> Locations;

            UPROPERTY()
            TArray<AText3DActor*> HintActor;
        protected:

            virtual void Activate();
        };

        // cpp中
        #include "RoutineGameplayTask.h"
        #include "Kismet/GameplayStatics.h"
        #include "Text3DComponent.h"

        URoutineGameplayTask* URoutineGameplayTask::ConstructTask(TScriptInterface<IGameplayTaskOwnerInterface> TaskOwner,TArray<FVector> locations)
        {
            URoutineGameplayTask* task = NewTask<URoutineGameplayTask>(TaskOwner);
            if (task) {
                task->Locations = locations;
            }
            return task;
        }

        void URoutineGameplayTask::Activate()
        {
            // 在指定位置打印提示字符
            int i = 0;
            for (auto& t : this->Locations) {
                ++i;
                auto* actor = GetWorld()->SpawnActor<AText3DActor>(t, FRotator::ZeroRotator);
                actor->GetText3DComponent()->SetText(FText::FromString(FString::Printf(TEXT("%d"),i)));
            }
        }
        ```

        所有在Actor中的修改
        ```cpp
        // h
        // ...
        class DEMOZERO_API AMyCharacter : public ACharacter, public IAbilitySystemInterface, public IGameplayTaskOwnerInterface
        {
            UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Ability" , meta = (AllowPrivateAccess = "true"))
            UGameplayAbilitySet* gameplayAbilitySet;

            UPROPERTY(EditAnywhere, BlueprintReadWrite,Category="Ability", meta = (AllowPrivateAccess = "true"))
            UAbilitySystemComponent* AbilitySystemComponent;

            UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Ability", meta = (AllowPrivateAccess = "true"))
            UGameplayTasksComponent* GameplayTasksComponent;	
            
            UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "EnhancedInput|Action", meta = (AllowPrivateAccess = "true"))
	        TObjectPtr<UInputAction> IA_BaseAttack;
           
            virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;
            virtual UGameplayTasksComponent* GetGameplayTasksComponent(const UGameplayTask& Task) const override;

            /** Get owner of a task or default one when task is null */
            virtual AActor* GetGameplayTaskOwner(const UGameplayTask * Task) const override;
	        void BaseAttack(const FInputActionValue& Value);
        }
        // cpp
        AMyCharacter::AMyCharacter()
        {
            AbilitySystemComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComponent"));	
            GameplayTasksComponent = CreateDefaultSubobject<UGameplayTasksComponent>(TEXT("GameplayTasksComponent"));
        }

        void AMyCharacter::BeginPlay()
        {
            // 初始化属性
            AbilitySystemComponent->InitStats(UCharacterBaseAttributeSet::StaticClass(), nullptr);

            // 创建GameTask测试
            URoutineGameplayTask* task = URoutineGameplayTask::ConstructTask(this, { FVector(0,0,100),FVector(1,3,100) });
            if (GameplayTasksComponent) {
                GameplayTasksComponent->AddTaskReadyForActivation(*task);
            }
        }

        UAbilitySystemComponent* AMyCharacter::GetAbilitySystemComponent() const
        {
            UE_LOG(LogTemp, Log, TEXT("from gett ability system component"));
            return AbilitySystemComponent;
        }

        UGameplayTasksComponent* AMyCharacter::GetGameplayTasksComponent(const UGameplayTask& Task) const
        {
            return GameplayTasksComponent;
        }

        AActor* AMyCharacter::GetGameplayTaskOwner(const UGameplayTask* Task) const
        {
            return const_cast<AActor*>(Cast<AActor>(this));
        }

        void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
        {
            Super::SetupPlayerInputComponent(PlayerInputComponent);
            // ...
            if (AbilitySystemComponent) {
                AbilitySystemComponent->BindToInputComponent(EnhancedInputComponent);
                if (gameplayAbilitySet) {
                    for (const FGameplayAbilityBindInfo& BindInfo : gameplayAbilitySet->Abilities) {
                        if (!BindInfo.GameplayAbilityClass) {
                            UE_LOG(LogTemp, Error, TEXT("GameplayAbilityClass %d not set"), (int32)BindInfo.Command);
                            continue;
                        }
                        FGameplayAbilitySpec spec(BindInfo.GameplayAbilityClass->GetDefaultObject<UGameplayAbility>(), 1, (int32)BindInfo.Command);
                        FGameplayAbilitySpecHandle abilityHandle = AbilitySystemComponent->GiveAbility(spec);
                        int32 AbilityID = (int32)BindInfo.Command;

                        FGameplayAbilityInputBinds inputBinds(
                            FString::Printf(TEXT("ConfirmTargetting_%s_%s"), *GetActorNameOrLabel(), *BindInfo.GameplayAbilityClass->GetName()),
                            FString::Printf(TEXT("CancelTargetting_%s_%s"), *GetActorNameOrLabel(), *BindInfo.GameplayAbilityClass->GetName()),
                            "EGameplayAbilityInputBinds",
                            AbilityID, AbilityID
                        );
                        
                        AbilitySystemComponent->BindAbilityActivationToInputComponent(EnhancedInputComponent, inputBinds);
                        AbilitySystemComponent->TryActivateAbility(abilityHandle, 1);
                        UE_LOG(LogTemp, Log, TEXT("AbilityId: %d"), AbilityID);
                        AbilitySystemComponent->AbilityLocalInputPressed(AbilityID);
                    }
                }
            }
        }

        void AMyCharacter::BaseAttack(const FInputActionValue& Value)
        {
            if (AbilitySystemComponent) {
                AbilitySystemComponent->AbilityLocalInputPressed(0);
            }
        }
        ```
## 网络通信
1. 常规通信
    - UE对HTTP、UDP、WebSocket都给予了支持，下面可见代码示例
        ```cpp

        ```
2. 游戏同步
    1. 概述：UE内部实现了TCP、UDP、HTTP、Websocket等协议。但如果打算使用UE自带的Dedicated Server进行服务器端开发，需要自行从源代码编译UE。
    2. 原理：

