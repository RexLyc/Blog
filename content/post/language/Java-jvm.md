---
title: "Java系列：JVM篇"
date: 2024-03-18T10:11:23+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
draft: true
---
本篇记录Java的JVM的一些设计、实现细节，对程序开发和面试有一定帮助。
<!--more-->
## 内存模型
### 引用
在JVM中，除了栈上的变量，其他的对象都存储在堆中，对象以引用的方式被使用。

函数传递都是传引用。赋值也是改变变量内所存储的引用。

### 垃圾回收
JVM历史上有多个垃圾回收算法。其中广泛使用的是，Parallel GC和CMS，G1，ZGC。从Java标准来看，Java8中主要使用的是PGC和CMS，Java21之后应当使用ZGC（在hotspotVM中）。

![10种Java垃圾回收器](/images/JavaSeries/all-gc.png)

图中的虚线代表配对组合。在更新的JVM种，分代内存模型已经废弃，都转变为了分区模型（小页面、中页面、大页面）。

从缺点来看，Paralle GC和G1都是百毫秒级，而CMS则没有解决碎片化的问题。

有一些术语值得学习：[记忆集和卡表](https://developer.aliyun.com/article/1097566)

这些方法的主要算法，都是基于标记-清理-复制算法。具体可以分为三个阶段：
- 标记阶段，即从GC Roots集合开始，标记活跃对象
- 转移阶段，即把活跃对象复制到新的内存地址上
- 重定位阶段，因为转移导致对象的地址发生了变化，在重定位阶段，所有指向对象旧地址的指针都要调整到对象新的地址上

在具体实现过程中，都会加入并发的手段，以减少STW时间。在G1算法中，标记阶段也会被分为三个阶段
1. 初始标记阶段：初始标记阶段是指从GC Roots出发标记全部直接子节点的过程，该阶段是STW的。由于GC Roots数量不多，通常该阶段耗时非常短。
2. 并发标记阶段：并发标记阶段是指从GC Roots开始对堆中对象进行可达性分析，找出存活对象。该阶段是并发的，即应用线程和GC线程可以同时活动。并发标记耗时相对长很多，但因为不是STW，所以我们不太关心该阶段耗时的长短。
3. 再标记阶段：重新标记那些在并发标记阶段发生变化的对象。该阶段是STW的。

但G1的清理和复制阶段则都需要STW。而且这个阶段的耗时会随着对象的增多而线性增长。

可见停顿主要来自于清理和复制，而G1无法在这一步做更多的并发优化的原因，是无法确切定位对象位置。即并发转移中“并发”意味着GC线程在转移对象的过程中，应用线程也在不停地访问对象。假设对象发生转移，但对象地址未及时更新，那么应用线程可能访问到旧地址，从而造成错误。

> 在ZGC之前，垃圾回收所需要的信息，都存储在堆内的对象头内。

> 只有从堆中读取引用会发生这个问题，因为堆内存被GC改变了。

ZGC通过着色指针、读屏障、以及指针转发表（旧着色指针、新着色指针）解决了这一问题。首先需要理解ZGC中使用的对象指针。

| 0~41位 | 42~45位 | 46~53位 |
| --- | --- | --- |
| 保存对象信息（地址），寻址空间4TB | 三色标记（M0、M1、Remapped）以及是否需要Finalize| 保留18位 |

从设计上来看，着色指针就是对象的引用（存疑）。这是一个进步，能避免在堆内的对象上再保留头部的GC信息。在做可达性检验时，也只需要对所有的着色指针进行着色。

由于ZGC对指针进行了重新的定义，因此还需要依赖于虚拟内存映射机制。多个ZGC的指针，实际可能指向的是同一个物理地址。即同一块物理内存，可能同时映射为M0（Marked 0）、M1、Remapped。但是任意时刻，只有一个视图是有效的。

Remapped代表为1初始化状态（或者转移完成的干净状态），M0为阶段1（含义为从GC Roots可达），M1为阶段2（在并发标记阶段，如果发现仍有位于M0状态的指针，表示是上一次GC未完成转发表处理，会进行处理，并标记为M1）。

而ZGC 的读屏障是在指针加载的操作的时候，插入一段针对该指针的处理逻辑：
1. 如果指针指向已经被转移的对象，那么读屏障将修正该指针
2. 在标记阶段，如果该指针未被标记（由于并发标记可能未完成，此时可能仍是Remapped），那么读屏障将标记该指针（标记为M0，GC Root可达）
3. 在转移阶段，如果该指针指向需要转移的区域，那么该指针指向的对象将被转移，然后修正该指针。 

到此可以叙述ZGC的所有阶段
1. 初始标记：STW，只标记GC Roots直连内容
2. 并发标记
3. 再标记：STW，处理漏标问题，对并发标记阶段发生变动的指针进行处理
4. 并发转移准备：确定需要转移的页面
5. 初始转移：STW，只转移GC Roots直连内容
6. 并发转移：负责将对象内容转移，但是不负责指针，未修改的指针会在下一次GC时经由M0变为M1而得到处理。

ZGC也提供了多种触发GC的机制：分配时、分配速率、固定间隔、主动、外部触发、元数据区分配。

> ZGC并不是银弹，目前为止ZGC是单代垃圾回收器（2024-03-20），但其实分代ZGC也在开发中。在具体场景下还需要具体分析。


### 本地内存


## 参考
1. [新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
2. [深入解析ZGC垃圾回收器](https://www.cnblogs.com/booksea/p/17665685.html)
3. [中篇｜丝般顺滑！全新垃圾回收器 ZGC 原理与调优](https://juejin.cn/post/7031092689357504520)
4. [一文深入分析 ZGC](https://heapdump.cn/article/4867489)
5. [【2022年最新版】新一代垃圾回收器ZGC深度解析](https://www.bilibili.com/video/BV1xF411B7vZ)