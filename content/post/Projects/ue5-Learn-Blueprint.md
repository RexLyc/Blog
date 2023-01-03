---
title: "UE5学习：蓝图篇"
date: 2022-12-01T22:47:06+08:00
categories:
- 计算机科学与技术
- 游戏
tags:
- 游戏
- 个人项目
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/ue-logo.png
---
本文是学习使用蓝图方式建立一个游戏工程的教学视频的课程笔记。
<!--more-->
## 概述
&emsp;&emsp; 蓝图是UE5面向非程序员类型的开发者推出的GamePlay脚本系统，对于一些简单目标，蓝图的功能已经足够完成。

## 蓝图编程
- 蓝图类
    - 角色：用于编辑角色相关的逻辑
        - 双击进入编辑界面，视口用于显示角色模型，事件图表用于显示和编辑事件
    - 游戏模式
    - 关卡蓝图：用于编辑整个关卡都生效的一些逻辑
    - 窗口控件蓝图：添加一些显示到屏幕上的空间内容
- 变量（Variable）：和材质的参数化类似，蓝图中也允许对外暴露一些变量便于利用相同的蓝图类生成不同的Actor
    - 在左侧变量处点击添加按钮
    - 设置类型、设置可见性、编译
    - 从左侧拖动变量到事件图表中，获取变量是将变量输出，设置变量是为其输入
- 常用事件节点：
    - 输入类：键盘/鼠标/手柄按键（Key Event）、
    - 设置类：
        - 设置碰撞启用（Set Collision Enable）：设置为开启碰撞检测
        - 设置模拟物理（Set Simulate Physics）：设置为开启物理模拟
        - 设置物理混合权重（Set Physics Blend Weight）：用于设置骨骼动作和物理模拟的混合程度
    - 调试类：打印字符串
    - 运动类：
        - 添加Actor本地旋转（Add Actor Local Rotation）：刚体旋转
    - 事件类：
        - 组件重叠事件（On Component Begin Overlap）：Other XXX为对XXX进行判断，当确认是指定组件，才真正触发这次事件。
        - 事件Tick（Event Tick）：每一帧都会触发的事件，注意直接使用输出会导致事件逻辑**不具备帧率无关性**，想要帧率无关，需要使用delta seconds属性
    - 计算类：
        - 创建旋转体（Make Rotator）：创建一个旋转运算单元，可以输入到Actor本地旋转中
        - 乘法（Multiply）：对输入做乘法
    - 类型转换：例如组件重叠事件的Other Actor转型为创建的角色蓝图类，转型成功则触发事件，否则触发失败信号
    - 自定义事件：Custom
- 常用组件
    - Mesh：各种模型
    - XXX Collision：各种碰撞体，可以利用其提供的事件，做一些简单的交互逻辑

- 调试：先将蓝图运行起来，然后在蓝图编辑界面中，在运行按钮旁边就可以选择调试的对象
    > 注：当鼠标在主视口中，因为开始运行后消失，可以使用Shift+F1将鼠标恢复出来
- 其他蓝图类技巧
    - 一些关于场景控制、程序逻辑等方面的内容，可以整理放到指定的蓝图类中，作为变量进行调整。比如通过一个浮点数控制所有树木的摇晃细节。

## 全局后处理
1. 类型：PostProcessorVolume
1. 可调整内容：曝光、Bloom、镜头、颜色分级、渲染功能等等
1. 影响范围：
    - 影响相机、而且只影响相机位于Volume体积内时（除非设置了无限范围Infinite Extend）
    - 可以对全局光照函数进行配置，配置其细节水平（小物体阴影）、最终采集质量（阴影噪点等）

## 材质
1. 基本用法：拖动一个材质到静态网格上
1. 材质节点属性：
    - 基本颜色：基本纹理
    - Metallic：金属性质
    - 粗糙度：字面意思
    - Normal：法向贴图
    - 全局位置偏移：允许材质控制模型顶点移动（常用于植物随风摆动等特效）
1. 创建自己的材质：
    - 在Material文件夹内新建材质，双击进入编辑
    - 编辑节点，连接，取消连接（Alt并单击）
    - 点击“应用”，进行材质编译
1. 材质节点：非常复杂，由很多的种类，这里记录一些基础的
    - Constant3Vector：RGB颜色常用节点，输出到基础颜色
    - TextureSampler：纹理采样器，可以输出RGB到粗糙度、高光度、各向异性、自发光等等
    - LinearInterpolate：线性插值，用一种值去混合两种值，如和TextureSampler搭配制作锈迹效果
        - 线性插值的输入数据A和B必须有同样的维度
    - TextureCoordinate：纹理坐标，用于配置纹理在材质中的平铺密度
    - Multiply：令A、B输入相乘
    - FlattenNormal：法向贴图专用调节，用于控制法向的清晰度（法向痕迹是否明显）
    - StaticSwitch：布尔切换节点，提供选项来将两个输入选择一个输出
        - StaticSwitchParameter：参数化版本
    - LandscapeLayerBlend：地形材质图层混合，输入数量不固定，根据需要配置不同的地形的各层的材质
1. 创建实时材质
    - 从已有材质节点，创建材质实例
    - 在已有材质的控制节点中，选择需要的节点进行**参数化**，并进行应用
    - 在材质实例中修改对应的参数，将该实例拖拽到对应的模型上
    > 注：参数化的命名不应当重名，重名的元素会被视为同一个元素
1. LOD：Level of Detail
    - 用于优化模型的材质
    - 可能的问题：使用一些第三方模型时，在模型编辑界面中，能看到多级的LOD，但是这里面可能有一些材质绑定错误，要记得删掉。

## 纹理
1. 创建自己的纹理：
    - 根据需要，创建漫反射纹理、法线、金属度、粗糙度、等等基础纹理图片
    - 导入图片并检查必要的选项：sRGB、纹理组
        - 只有漫反射纹理需要开启sRGB，其他均**不应**开启sRGB，这里的原因主要是因为开启后会对图片颜色深度产生变化（GAMMA矫正），*这一点并不是十分理解*
    - 配置tile参数（平铺密度）
        - 添加Multiply、Texture Coordinate节点，并将其输入到各个TextureSampler的UVs中
1. 一些窍门：
    - 金属质感（Metallic）、和粗糙度（Roughnes）等纹理经常被融合到一张纹理图片中，并分通道使用，比如金属用R，粗糙度用G。
1. 坑：
    - 法向贴图的计算方式有OpenGL和DirectX两种，这两种互不兼容，Unreal是使用DX方式计算，Blender则是使用OpenGL方式。如果发生法向错误，可以去法向贴图中，选择**翻转绿色通道**。
## 模型
1. 文件类型和区别：
    - fbx：
1. 创建模型并绑定材质：
    - 导入模型

## 光照模型
&emsp;&emsp; 使用Lumen进行全局光照。Lumen是Unreal比较好的一个全局光照函数，效率高。在项目全局设置中可以设置渲染相关的内容，其中可以对Lumen实际使用的方式进行调整。
- 光源类型：
    - 定向光源：类似于太阳光，指定方向的光源
    - 点光源：向各个方向衰减
    - 聚光灯点光源：有一个光锥的点光源
    - 立方体光源：一个发光立方体
    - 天空光：将天空盒的颜色作为参照，给所有物体施加一个柔和的光照，需要有一个天空环境（SkyAtmosphere）
    > 注1：必须有定向光源（而且配置了天空大气选项）才会让天空盒能被绘制出来（因为其本身的光照效果是依赖于太阳这种定向光的）。
    > 注2：天空光，在配置了定向光源后，需要修改天空光为可移动，并将光源设置为实时捕获（因为天空光实际上是需要根据天空颜色实时更新的）
- 常用搭配的视觉效果
    - 天空环境：填充天空的上半部分
    - 指数级高度雾：填充天空的下半部分，注意配置起始距离以避免影响到场景模型的渲染
    - 后处理：全局光照模型有一些参数需要在后处理中配置，比如Lumen的最终采集质量，应当设置到较高，以避免渲染结果出现很多噪点
- 和烘焙灯光的对比
    - 烘焙效果可以更好，运行时更快，但是相对的，需要更大的程序体积，以及开发过程中更多的用时，以及相对静态的灯光和物体
    - Lumen等实时全局光照效果，运行时相对慢，但是可以支持完全动态的灯光和物体，体积相对小
- 烘焙灯光用法
    - 将所有灯光设置为固定（不可移动），在后处理中将全局光照方法设置为空，反射方法设置为空
    - 构建→构建所有关卡（原文是Build All Levels），此时将会得到一个非常差的全局光照效果
    - 一些基本的优化思路：
        - 提高渲染计算质量：比如提高世界场景设置（World Settings）中的光照Lightmass设置中的反射数、反射质量和间接光照质量A，同时降低静态光照等级范围B，注意A×B需要等于1
        - 提高光照贴图的存储密度：对每一个模型，修改其细节设置中的光照Lightmass设置中的“覆盖的光照贴图分辨率”，提高该值以获得更高精度的光照贴图LightMap
        - 去除一些不必要的计算：关闭后处理中的环境光遮蔽计算（Ambient Occlusion）
        - 增加反射捕获：添加视觉效果→反射捕获，移动到合适的位置，构建反射捕获

## 地形编辑
- 基本内容
    - 创建：初始化地形大小、分辨率
    - 管理：管理地形中的各个地块
    - 雕刻：对地形进行雕刻
    - 绘制：对地形表面进行材质绘制
- 步骤
    - 进入地形模式，在左侧设置地形参数，参考[4.27版本官方推荐配置](https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/Landscape/TechnicalGuide/)，点击创建
    - 一般会先回到选择模式，配置一下光照，便于观察
    - 回到地形模式，使用雕刻画笔，绘制地形
- 准备地形材质
    - 一般来说是使用LandscapeLayerBlend混合多种材质，以用于不同的地形的纹理
    - 地形材质不能直接拖拽材质球到主视口中，需要拖拽到右侧细节设置中的地形材质选项
    - 在地形模式的绘制菜单中，为不同层的材质创建图层信息，根据需要选择（一般权重即可）
    - 地形的纹理平铺尺度需要借助一些内容来进行参考，一般可以使用添加→添加内容或功能包→添加第三人称→使用里面的人物模型
    > 注：厉害的地形纹理可以做到自动化，即根据地形特点，自动输出不同的层，比如平地输出草地，高度变化很大的地方输出为岩石材质
- 地形编辑是一个很复杂的事情，可以再多看看相关教学视频

## 植物编辑
- 基本步骤
    - 切换到
    - 添加植物素材：下载带有Foliage功能的素材（会自动添加），或者将自己的素材手动拖动到Foliage素材列表中
    - 勾选想要使用的素材，左键绘制、Shift+左键删除
- 一些选项：
    - 过滤器：设置Foliage会在哪些类型的模型上生成植物
    - 选择好要生成的植物后注意在搜索框旁边有一个编辑按钮，可以具体控制植物生成细节
- 坑：
    - 树木使用植物编辑时要小心，因为可能瞬间加入大量树木，直接卡死主机

## 素材
- 素材库
    - Mega Scan：一个UE官方制作的库，也是免费使用，有大量的素材可以使用，
        - 入口：内容浏览器→添加→添加Quixel内容。或者从添加里也有Quixel Bridge。
        - 推荐：Trees、Cliffs
    - Epic Games Marketplace：Epic官方的商城，也有一些免费的素材
- 其他推荐
    - temperate vegetations
## 其他常用添加
- 玩家出生点
- 

## 其他特性
- Nanite：一个大幅度优化密集多变形场景绘制的系统。目前对静态网格体效果最好，比如岩石、戈壁场景，使用Nanite优化的模型能够大幅度提升性能。在网格体的细节窗口设置中开启

## 一些小技巧
1. 在任何节点图中，按Alt抗原查看节点的原始英文名称。
1. 调试时可以使用反引号按键呼出脚本菜单，一些常用的控制指令如
    - 控制锁帧30：t.maxFPS 30
1. 水的制作：基本上就是制作一个平面，给这个平面合适的材质即可
1. 编辑场景可以使用无光照等模式，提高运行速度

## 参考
1. [Unreal Engine 5 Beginner Tutorial - UE5 Starter Course 2022](https://www.youtube.com/watch?v=k-zMkzmduqI)