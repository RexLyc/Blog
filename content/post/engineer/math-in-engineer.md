---
title: "工程中的数学：合集"
date: 2024-05-14T14:40:52+08:00
categories:
- 理论
- 数学
tags:
- 工程实现
- 数学
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/math-in-engineer.png
draft: true
math: true
---
对于一些工程中可能用到的数学知识，在此进行记录。
<!--more-->
## 变换
### 傅立叶变换
傅立叶变换的理解并不容易。这里非常推荐观看3Blue1Brown的相关视频，参考[形象展示傅里叶变换](https://www.bilibili.com/video/BV1pW411J7s8/)。

其中最重要的理解就是傅立叶变换从结果上看，就是将时域的图像，尝试用不同的频率环绕在复平面的单位圆上。而积分的过程，可以认为是在求这个环绕之后的图形的质心。

当某一种频率的环绕方式，能够最大程度的契合原始的时域信号时，此时的质心将会达到最大的模长（变化的过程就是在求原始信号和基，这两部分的乘积）。表现上就是频域上的一个波峰。

$F(f)(w) = \int\limits_{-\infty}^{\infty}f(t)e^{-iwt}dt$

但是这个公式显然在实际应用中是无法计算的。因此需要由DFT（离散傅立叶变换）、FFT（快速傅立叶变换）。这一步推荐参考[DFT由来之路](https://www.bilibili.com/video/BV1kg4y1U7gc)、[双语字幕-快速傅里叶变换(FFT)](https://www.bilibili.com/video/BV1sE421K7XK)。

首先需要对连续函数做离散化。这一步是比较明显的。将$f(t)$和$iwt$都进行离散化。

$\hat{X}[k]=\sum_{0}^{N-1}X[n]e^{-i\frac{2\pi}{N}nk}$。现在的频率是$k$了。

这个时候从计算上来看，是一个大矩阵的运算（若干个$X[n]$和若干个旋转）


应用举例：
1. 大数乘法：参考[高精度乘法运算](https://www.bilibili.com/video/BV1gY411d7Z1)。


### 小波变换
小波变换的出现是为了解决傅立叶变换的一些局限性。例如
1. 傅立叶变换只给出了时域到频域的转换。但是某一个**频率在何时出现**是不知道的。
2. 傅立叶变换在一些方形波的情况下，不能很好的拟合。而这种信号在数字信号中非常常见。


参考[如何通俗地讲解傅立叶分析和小波分析间的关系？ - 咚懂咚懂咚的回答 - 知乎](https://www.zhihu.com/question/22864189/answer/40772083)