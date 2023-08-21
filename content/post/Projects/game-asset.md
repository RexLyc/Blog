---
title: "游戏开发：资产篇"
date: 2023-07-30T20:07:00+08:00
categories:
- 游戏开发
tags:
- 系列开坑
- 游戏开发
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/game.jpg
draft: true
---
本文章记录游戏开发中所有的数字资产的制作方式。尽量做到便宜好用。
<!--more-->

## 美术
1. 2D
2. 3D
   1. Blender（3.6.1）：[操作手册](https://docs.blender.org/manual/zh-hans/3.6/index.html)
      1. 常用操作
         | 操作名称 | 快捷键 | 注意 |
         | --- | --- | --- |
         | 移动旋转缩放 | G、R、S | 输入后再按X、Y、Z可以固定轴 |
         | 放大缩小相机 | Ctrl + 鼠标中键并滑动鼠标 | |
         | 复制 | Shift + D | |
         | 旋转视口 | 鼠标中键 | 按住Alt可以吸附旋转到一些固定角度！ |
         | 移动视口 | Shift + 鼠标中键 |  |
         | LoopCut（增加环） | Ctrl + R | |
         | PropotionalEdit（衰减编辑） | 先按O，再进行其他变换 | 将一个变换施加给附近元素，类似于PS的液化 |
         | 切换所选元素级别 | 1、2、3 | 分别是点、线、面 |
         | 切换模式 | Tab | 编辑模式、物体模式 |
         | 切换模式（轮盘） | Ctrl + Tab | 会呼出一个轮盘，选择所有的六种模式 |
         | 面挤出 | E | |
         | 独立挤出 | Alt + E |  |
         | 添加元素 | Shift + A |  |
         | Apply应用（变换） | 选择元素后Ctrl + A | 选择需要的变换（如Visual Geometry To Mesh，应用所有Modifier） |
         | 合并元素（物体） | Ctrl + J | 将右侧Collection中的元素合并 |
         | 分离元素（点线面） | 选中后按P | 可根据选择、材质等分离 |
         | 吸附 | Shift + Tab（或顶部磁铁按钮） | 可选择吸附目标 |
         | 选出循环元素 | Alt | 按住Alt时为loop select（循环边、面） |
         | 切割 | K | 用来在面上切割出一个新的面 |
         | 3D游标设定 | Shift + S | 选定一些点后，按住并选择需要的3D游标计算方式 |
      2. 编辑模式
         | 操作名称 | 快捷键 | 备注 |
         | --- | --- | --- |
         | Poly Build |  | 利用附近顶点创建多边形（Ctrl创建新的，Shift删除已有的），设置自动合并阈值更方便 |
         | Add | F3 | 自动添加各类东西（**非常实用**） |
         | Add（Grid Fill） | F3 | 对**偶数顶点**的循环边，自动填充空洞 |
         | Add（Bridge Edge Loops） | F3 | 对两个循环边，自动点对点补充桥面（适合和内插面配合来挖洞） |
         | Fill创建面 | F | 选中指定的边、点 |
         | 重新计算法向 |  Shift + N | 选中指定的面，多用于翻转个别错误法向 |
         | 挤出（Extrude） | E | 从一些选定点挤出多边形 |
         | 内插面（Inset）| I | 从一个选定面，向内插出一个面（内插后面数变为原来的边数+1） |
         | 倒角（Bevel） | Ctrl + B | 对一个选定边进行倒角 |

      3. 雕刻模式下操作
         | 操作名称 | 快捷键 | 注意 |
         | --- | --- | --- |
         | 调整衰减编辑区域大小 | 按住F并左右移动鼠标调整 | |
         | 调整衰减羽化 | 按住Shift + F并左右调整鼠标 |
         | Clay粘土 | C | 增减模型（圆形笔） |
         | Clay Strip粘土条 |  | 增减模型（方形笔） |
         | Smooth平滑 | Shift + S | 使模型平滑 |
         | Grab抓取 | G | 拖拽模型，按Ctrl是沿着法向移动 |

         > 注1：大部分工具，同时按下Ctrl可以使效果反向（隆起/凹陷）、Shift则是平滑

         > 注2：推荐学习视频：[【中字】Blender雕刻基础知识 / 雕刻快速入门](https://www.bilibili.com/video/BV1tD4y1X7zi)
      4. Modifier说明
         | Modifier名称 | 作用 | 注意 |
         | --- | --- | --- |
         | Mirror | 创建镜像元素 | 可以按照轴，也可以按照一个物体对象镜像 |
         | Subdivision Surface | 细分表面 | 原有物体的点成为控制点，可以进一步通过LoopCut增加控制点 |
         | Remesh | 重新网格化 | 根据给定算法和参数重新计算物体网格，**注意计算量** |
         | ShirnkWrap | 收缩 | 约束对象顶点并移动到目标对象表面（可以和吸附模式一同工作） |
         | Multriesolution | 多级精度修改器 | 对不同细分级别下的物体模型进行修改 |
         
         > 注：修改器先后顺序是有影响的
         
         > 注：收缩包裹的若干种模式中，最近点和投影最常用，后者能保证一定对称

      5. 建模工作流：
         - 对称建模（镜像修改器、镜像操作）：
           1. 人体：
              1. 先用立方体和圆柱体，搭配细分修改器，构建起大概的人体框架（主要骨骼和肌肉）
              2. remesh（这一步不是严格对称的），并进行雕刻，这一步要雕刻出几乎完整的人体轮廓（这一步面数会非常多）
              3. 重拓扑，目标是获得干净（面数合理）且有质感的模型。注意重拓扑先后进行**两次ShrinkWrap**，并适当使用Multiresolution细分。
              4. 制作基本的衣物（注意仍然可以使用ShrinkWrap缩裹，搭配Solidify、Subdivision，来获得紧贴身体的衣物效果）
      6. 经典问题：
         1. 建模的过程中忘记镜像，导致物体重拓扑时有很多问题？
            可以和一个大立方体作差，保留一半，重新镜像
      7. 经验教训：
         1. 使用表面细分时，应当保持控制网格都是四边
         2. 对于会弯曲的位置，要增加细分
         3. 手指、脚趾，一开始就要分开，否则重拓扑时非常麻烦
         4. 切换物体进行雕刻等模式操作时，先将当前物体退出到**Object Mode**
         5. 在合适的时候，Object模式下切换平滑光照和flat光照，能够更好的看出拓扑问题。
         6. 耳朵面部不应在重拓扑之前雕刻过多细节，会导致重拓扑变得非常困难。
         7. 在合适的情况下，为顶点分组
         8. 编辑软体结构，用粘土（C）雕刻效果往往优于拖拽（G）
         9. **多看多练**

         > 在Blender的任意位置，都可以通过F1获取帮助文档
      8. 界面特别说明
         1. 中间正上方：包含各类吸附、画笔衰减、轴心点选择
            ![各种吸附、3D游标的选择位置](/images/gameassets/blender_cursor_etc.jpg)
## 音乐
1. 代码式
   1. SuperCollider
      1. 参考：
         - [Github: awesome SuperCollider](https://github.com/madskjeldgaard/awesome-supercollider)
   2. Sonic Pi
      1. Windows10：安装后无法启动
2. DAW
   1. CakeWalk：
      1. 问题：创建项目卡死，可以创建完全空白的，其他任何模板均会卡死。
   2. WaveForm:
      1. 收费软件的免费版本，有一些限制，但不影响使用。
      2. 不能直接和Minilab联动
3. 音源软件
   1. Kontakt：不打开DAW，单独使用时没有声音
   2. 优先找一些免费的：
      1. [SOUNDS OF CHINA SAMPLE PACK | ROYALTY FREE TRADITIONAL CHINESE INSTRUMENTS](https://www.youtube.com/watch?v=K3vjNtKDXeU)

> 参考：
> - [LANDR Blog：The 12 Best Free DAWs to Create Music](https://blog.landr.com/best-free-daw/)
> - [Best Free DAWs for Windows in 2023: Make Music on a Budget](https://www.youtube.com/watch?v=Un6xUqBWSe0)
> - [LIKEMUSIC 音频术语教程](https://www.bilibili.com/video/BV1ye4y1T7Z3)
> - [023 Cakewalk by BandLab 基础教程](https://www.bilibili.com/video/BV1dt4y1J7pT)
> - Asio4All、Asio Link Pro

## UE插件
1. FluidFlux：流体插件

## 资源网站
1. [ANATOMY360：一个提供精细三维重建模型的网站](http://anatomy360.info/)
1. [【教程】Blender + UE5 游戏角色建模材质绑定动画全流程](https://www.bilibili.com/video/BV1MY4y1X7gn/)
1. [【中文字幕】UE5与Blender完整游戏环境制作工作流程视频教程](https://www.bilibili.com/video/BV1Ft4y1T7KW)
1. [Blender学习笔记：一个Q版人物](https://space.bilibili.com/27462787/channel/collectiondetail?sid=902549)