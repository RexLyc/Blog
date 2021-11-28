---
title: "Photoshop：多照片合成"
date: 2021-11-28T15:59:45+08:00
categories:
- 实用工具
- 平面后期
tags:
- 实用工具
- 摄影
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/PS.jpg
---
在HDR、全景图片等场景中，使用多张照片合成最终的输出结果是非常常见的操作。本文讲解使用Photoshop & Camera Raw进行合成的办法。
<!--more-->
# HDR
<center><img src='/images/Photoshop/ACR_HDR.png'>HDR生成示例</center></br>

1. HDR：高动态光照渲染。High-Dynamic Range。
1. 为什么需要：人眼是高动态范围的，就是说在一个有很亮物体和很暗物体的场景中，我们依然能够相对良好的观察画面中高光、阴影处的细节。但是相机做不到。所以需要采用多张照片，使用不同的曝光参数，最终合成一张，同时具有高光细节和阴影细节的图片。
1. 注意的细节：最少2个，一般3个。根据环境的情况决定。
1. 做法（Camera Raw）：
    1. 导入要生成HDR照片的多张输入照片
    1. 全选，右键合并到HDR

# 全景图
1. 目标：当相机焦段无法涵盖全部的取景范围，使用多张图片，后期合成一张全景图
1. 注意的细节:
    - 两张图片之间应当有30%的重叠范围。
    - 相机拍摄时的各种参数应保持完全一致。也因此建议使用手动档
1. 做法（Camera Raw）；
    1. 导入
    1. 全选，邮件合并到全景图
