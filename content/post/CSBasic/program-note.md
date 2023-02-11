---
title: "编程题笔记：总结篇"
date: 2022-03-29T10:28:30+08:00
categories:
- 求职
- 习题
tags:
- 持续施工
- 习题
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algo-exercises.jpg
math: true
---
本篇以记录实用算法和技巧为主，和题目记录对照使用。
<!--more-->
## 二分
- 参考STL实现：[lower_bound](https://zh.cppreference.com/w/cpp/algorithm/lower_bound)、[upper_bound](https://zh.cppreference.com/w/cpp/algorithm/upper_bound)

## Partition
- 参考STL实现：[partition](https://en.cppreference.com/w/cpp/algorithm/partition)

## 位运算技巧
- 位运算不保证跨平台通用，实际生产中使用时需要注意
- 无临时变量交换两个数的值：异或^
- 二进制下统计1的个数、去掉最低位的1：n&(n-1)
- 查表法：统计1的个数（分四个字节）、计算奇偶校验位
- 子集枚举
- 寻找一个/两个仅出现一次的数（其余两次）：异或^
- 更多技巧请参考：[位运算](https://blog.csdn.net/deaidai/article/details/78167367?utm_source=distribute.pc_relevant.none-task)、[位运算奇技淫巧](https://blog.csdn.net/holmofy/article/details/79360859)。

## 弗洛伊德判圈法
- 说明：一个可以在有限状态机、迭代函数、链表等算法或数据结构中判断是否存在环，以及判断环起始位置、长度的算法
- 原理：用两个以不同速度前进的指针遍历，如果两个指针在结束前相遇，则必定存在环。此时分别在初始位置、相遇位置额外构造2个指针，以相同速度遍历，这一次相遇点一定是环的入口处。
- 参考证明：[龟兔赛跑算法](https://blog.csdn.net/mq2553299/article/details/104045740)

## 分割平面与空间公式
- 直线划分平面：已有n-1条直线，新增第n条直线和n-1条线最多形成n-1个交点，这些焦点将新增直线划分为2个射线和n-2个线段，该直线外侧即本次新增直线后，新增加的平面区域数量，恰好和射线、线段一一对应，因此有：
    $f(n)=f(n-1)+2+(n-2)=f(n-1)+n,f(1)=2$
- 折线划分平面：一个折线是形如五角星的一个角（如$<$）。已有n-1条折线，新增第n条折线，和n-1条折线（攻击2*(n-1)条射线），最多形成4*(n-1)个线段（折线顶点处不再有射线），和2个射线，但折线顶点处多算了一个，因此有：
    $f(n)=f(n-1)+4(n-1)+2-1=f(n-1)+4n-1,f(1)=2$
- 平面分割空间：分割最大化时，相当于每一个平面需要和之前的所有平面相交，此时我们可以想象到，该平面上将会被若干交线划分为若干平面区域，该区域数量就恰好是新增的空间数量，联系前述公示，可有：
    $h(n)=h(n-1)+f(n-1)$

## Morris遍历和线索二叉树
- 参考：https://blog.csdn.net/softwarex4/article/details/95986102 ，https://www.jianshu.com/p/d2059062efac

## 前缀树：
- 一个小技巧是给字符串添加一个终止字符’$’，便于标记字符串的终止。
- 参考：[前缀树(字典树、Trie树)](https://www.cnblogs.com/zhouzhiyao/p/12547142.html) 。

## 后缀树：
- 讲的比较好的几篇：https://www.cnblogs.com/xubenben/p/3484988.html 、 https://www.cnblogs.com/xubenben/p/3486007.html 、 https://blog.csdn.net/aiphis/article/details/48489709 、https://www.cnblogs.com/gaochundong/p/suffix_tree.html。
- 几个关键点:
    - 活跃节点的理解（当前能够重叠的部分的起始位置）
    - 活跃半径（重叠长度）
    - 活跃边（可能没有，或者就是重叠的起始字符
    - 每次的三个规则
- 参考：[后缀树详解及其具体应用](https://blog.csdn.net/Yuzhiyuxia/article/details/24305683)、[后缀树Ukkonen构造法](https://blog.csdn.net/smbroe/article/details/42362347)、[后缀树系列一:概念以及Ukk实现原理](https://blog.csdn.net/fjsd155/article/details/80211145)

## 后缀数组：
- 实现起来比较好理解，而且速度也不慢（不过一般需要引入额外的信息来解题）。https://www.cnblogs.com/jianglangcaijin/p/6035937.html 这里有一篇很好的博客。DC3算法是比较好的实现方式（比较严格的O(3n)），对基数排序的理解。尤其是字符串的特点，导致划分完的2各部分，内部一定是有序的（长度都不一样，不可能出现完全相等）。而网上的对比可以看一下，起始倍增法也算够优秀（实际运行速度）。
- 参考：[后缀数组解析及应用](https://blog.csdn.net/yxuanwkeith/article/details/50636898?_=_)

## 线段树
- 参考：[史上最详细的线段树教程](https://zhuanlan.zhihu.com/p/34150142)

## RMQ区间最大值最小值查询
- Tarjan的Sparse-Table算法
    - 构建：$dp[i][j]$表示i为起点，长度为$2^j$的区间的最值。状态转移方程（可以理解为二分）为$dp[i][j]=min/max(dp[i][j-1]+dp[i+(1<<(j-1))][j-1])$。注意二层循环，j为外层（按照区间长度分治）。
    - 查询：寻找小于等于区间范围的区间（或多个区间，可以重叠）。
- 线段树：$O(n)$建立，$O(logn)$查询
- 笛卡尔树+LCA+DP：$O(n)$建立，$O(1)$查询

## 树状数组
- 参考：[树状数组详解](https://www.cnblogs.com/xenny/p/9739600.html)

## 其他树：
- 参考：[二叉树最近公共祖先（LCA）详解](https://www.hrwhisper.me/algorithm-lowest-common-ancestor-of-a-binary-tree/)

## 图论
- 参考：[图算法总结](https://www.cnblogs.com/Ace-Monster/p/9439557.html)

## 字符串匹配：
- 总结：
    - KMP：构造next数组，最长的相等前缀、后缀
    - BM：过于复杂，而且很长
    - Sunday：启发式，好写又好用，两个条件：适配时先看目标串的下一个待匹配字符，如果不存在于模式串中，则直接大跳，否则移动模式串到最右侧第一个能匹配到该待匹配字符的位置。
- 参考：
    - [BF、KMP、BM、Sunday详解](https://www.cnblogs.com/Syhawk/p/4077295.html)
    - [从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)
    - [BF、RK、BM、KMP、Trie树、AC自动机](https://blog.csdn.net/weixin_40805537/article/details/89044710)

## 回文字符串Manacher算法：
- 概述：$O(n)$时间求字符串的全部回文子串
- 核心原理：当前中心C，右边界R，待求$p[i]$，关于C镜像$p[2*C-i]$，只有当镜像回文长度会超出当前C的回文的左边界，或者直接赋值镜像p回文长度会超过右边界R，才需要中心扩展法
- 参考：[马拉车算法详解](https://zhuanlan.zhihu.com/p/70532099)

## 正则表达引擎
- 参考：[正则表达引擎的原理](https://www.cnblogs.com/snake-hand/p/3153396.html)

## 大数乘法：
- 小于10w数据可以使用竖式乘法，大于10w可以使用快速傅里叶变换(FFT)
- FFT关键词：
    - 参考：[MIT公开课（分治法和FFT）](https://www.youtube.com/watch?v=iTMn0Kt18tg) 、[博客园-快速傅里叶变换（FFT）详解](https://www.cnblogs.com/wangyh1008/p/9325715.html)、http://picks.logdown.com/posts/177631-fast-fourier-transform、以及[一个比较好的迭代](https://leetcode-cn.com/problems/multiply-strings/solution/fftjie-fa-by-kuai-xue-shi-qing-2/)
- 参考：[大数乘法高效算法](https://blog.csdn.net/u010983881/article/details/77503519)

## 海量数据处理
- 参考：[海量数据处理面试专题](https://blog.csdn.net/v_july_v/article/details/6685962)

## 语言相关要点
1. C/C++：
    1. IEEE754数字标准在C++的实现：[传送门](https://zh.cppreference.com/w/cpp/language/types)
    1. 整数不加L后缀，默认为int
        ```cpp
        long long bigNum = 1 << 32; // 报错
        ```
    1. ""的字面值类型，是const char *，无法直接用于字符串拼接运算
    1. 可以同一行定义多个变量
        ```cpp
        int a = 0, *aa = nullptr, b = 1; // 可以但不好看
        ```
    1. 求值顺序：最简单的办法就是多用括号，并且将运算分多行来写。[参考文档](https://zh.cppreference.com/w/c/language/eval_order)

## 一些优化思路
1. 用计算代替分支：分支跳转的代价较高。
1. 尽量将变量定义放在循环外。
1. 多使用位运算。
    - 除2，等价于右移>>1
    - 乘2，等价于左移<<1
    - %2，等价于&1
1. 用数组代替哈希表
1. 2维数组展开到1为手动计算下标

## 经典书籍
1. 《程序员面试经典》
    - 08.07，全排列的生成：可以去看[C++的next_permutation的实现](https://zh.cppreference.com/w/cpp/algorithm/next_permutation)
    - 16.01，无临时变量交换数字：异或
    - 08.04，幂集：使用二进制位运算
    - 08.05，递归乘法：不用*做乘法，用递归
    - 16.20，键盘T9：字符串题目，可以学学前缀树
    - 04.08，首个共同祖先
    - 16.03，数学题：求线段交点
    - 08.11，硬币：实际上是无限背包问题
1. 《剑指Offer》
    - 题4，二维有序数组：背一下C++的[二分查找实现](https://zh.cppreference.com/w/cpp/algorithm/upper_bound)
    - 题7，重建二叉树
    - 题10，fibo数列：常规矩阵快速幂
    - 题11，旋转数组的最小数字。
    - 题18：删除链表的节点，使用**指针。
    - 题64：求1+2+...+n，不让用各类控制语句。要结合递归和短路判断来完成。
    - 题56-2：数组中数字出现次数II。位运算和状态转移。**很好的题目，值得二刷**。
    - 题19：正则表达式的匹配，经典dp。
    - 题49：丑数，经典递推题目。
    - 题44、43：数字序列中的某一位；1~n整数中1出现的次数。都是观察数字规律类型的题目。
    - 题41：数据流中的中位数。经典的双堆的使用。
    - 题59-2：O(1)时间实现队列，并能返回当前最大值。用一个单调队列（双端）+普通队列实现。**很好的题目，值得二刷。**
    - 题62：约瑟夫环问题。找映射关系。可以看一个[相关leet-code题解](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-by-lee/)。

## 参考
1. 《编程之法：面试和算法心得》：尤其里面的海量数据处理一章值得一读。
1. 《算法导论》：经典中的基础
1. [知乎：有哪些学习算法的网站推荐](https://www.zhihu.com/question/20368410/answer/954418248)
1. [July《十五个经典算法研究与总结》索引](https://blog.csdn.net/sailinglt/article/details/79323509)
1. [国家队资料](https://github.com/enkerewpo/OI-Public-Library)