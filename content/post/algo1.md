---
title: "算法导论其一（施工中）"
date: 2021-07-08T22:41:56+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
keywords:
- tech
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
工欲善其事必先利其器，写出优良算法的基础是拥有对算法性能衡量的能力。一起回顾一下时间复杂度的计算吧。
<!--more-->
> 假设计算机无限快，并且计算机存储器是免费的，那么你还有任何理由来研究算法吗？
> <p align="right">——《算法导论》</p>
**https://note.qidong.name/2018/03/hugo-mathjax/ 提醒一下自己公式绘制问题还没解决呢**
# 关注点
0. 性能：尽量跑的快一点
1. 安全：比如不要泄露内存
2. 扩展：泛用性很强（对字符串排序、对数字排序）
3. 易用性：接口简单
4. 鲁棒性：稳定（不要忽快忽慢，忽然崩溃）
# 性能衡量工具
0. 数学符号
    - ~~$\sum_{i=0}^N{X_i}$~~
1. 主方法（Master Method）
2. 画递归树
3. 代换法（猜想并证明）
4. 举例
# 平摊分析
部分算法、数据结构，它的单一操作可能最坏、但平摊耗费较小。不保证实时性能。对于这类算法和数据结构，不能用常规方法分析，需要对其平均性能进行估计。

0. 例子
1. 聚集分析
2. 记账法
3. 势能法
# 竞争分析
0. 分类
1. 应用
2. 方法
3. 举例
# 参考资料


*学点英语，主要是尽量把术语发音正确噢* :winking_face_with_tongue:<a href="https://www.bilibili.com/video/BV1Tb411M7FA?from=search&seid=3716615071312119347" target="_blank">MIT6.046Jのbilibili传送门</a>
# 本节博客搭建过程中的坑
0. 添加内嵌html支持，需要修改toml，令markdown引擎支持内嵌html（unsafe模式）
1. <a href="https://note.qidong.name/2018/03/hugo-mathjax/" target="_blank">使用MathJax在Hugo的Markdown中绘制公式</a>
    - 第一步：添加一个单独的html于主题文件夹partials下，并引入所需的js库，css内容。
    - 第二步：去head.html中利用插值表达式引用该html。
    - 第三步：最好是在markdown开头的Front Matter部分添加math: true。也可以在toml中全局启用MathJax。