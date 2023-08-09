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
      2. 编辑模式
         | 操作名称 | 快捷键 | 备注 |
         | --- | --- | --- |
         | Poly Build |  | 利用附近顶点创建多边形（Ctrl创建新的，Shift删除已有的），设置自动合并阈值更方便 |
         | Add | F3 | 添加各类东西 |
         | Add（Grid Fill） | F3 | 对**偶数顶点**的循环边，自动填充空洞 |
         | Fill创建面 | F | 选中指定的边、点 |
         | 重新计算法向 |  Shift + N | 选中指定的面，多用于翻转个别错误法向 |

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

      5. 建模工作流参考：
         - [【教程】Blender + UE5 游戏角色建模材质绑定动画全流程](https://www.bilibili.com/video/BV1MY4y1X7gn/)
         - [【中文字幕】UE5与Blender完整游戏环境制作工作流程视频教程](https://www.bilibili.com/video/BV1Ft4y1T7KW)
         - [Blender学习笔记：一个Q版人物](https://space.bilibili.com/27462787/channel/collectiondetail?sid=902549)
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
      > 在Blender的任意位置，都可以通过F1获取帮助文档
## 音乐
1. SuperCollider
   1. 参考：
      - [Github: awesome SuperCollider](https://github.com/madskjeldgaard/awesome-supercollider)
2. Sonic Pi
   1. Windows10：安装后无法启动

## 资源网站
1. [ANATOMY360：一个提供精细三维重建模型的网站](http://anatomy360.info/) 