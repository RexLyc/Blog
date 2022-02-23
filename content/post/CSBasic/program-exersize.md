---
title: "编程题笔记"
date: 2021-12-30T16:29:39+08:00
categories:
- 求职
- 习题
tags:
- 持续施工
- 习题
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algo-exercises.jpg
math: true
---
题目内容来自leetcode、牛客等平台，偶有面试笔试真题。
<!--more-->
# 总结
1. 惯用语言为C++，偶尔会使用Java、Python。
1. 由于牛客等平台在面试时，不一定会提供头文件，需要自己牢记使用的标准库函数位于哪个头文件中。
1. 会给出一些我觉得非常好看的题解、总结博客的链接。
1. 算法分类：
    1. 搜索（BFS、DFS）：单目标搜索、多目标搜索，搭配数据结构优化，单调性优化
        - 搜索从思路上可以分为：生成法、判别法两种。前者生成的一定是一个解，后者生成所有，并进行判定。
    1. 动态规划：记忆化的搜索。
        1. 单状态dp[][][]，多状态dp1[]、dp2[]
        1. 多种决策可能
        1. 递推形式
        1. 区间dp，最优化区间
        1. 背包问题
        1. 空间优化，[时间优化参考](https://www.cnblogs.com/flipped/p/9669202.html)
    1. 贪心
    1. 分治：可分性、合并时的复杂度
    1. 排序：排序后是否可贪心
    1. 数据结构
        1. 单调栈、单调队列
        1. 双堆
        1. 数组：双指针、预处理、和递归结合
        1. 树：前中后序的递归&非递归写法，层次遍历
    1. 图算法：并查集、拓扑排序、欧拉图、最短路、网络流
        - 偶尔会有字符串等其他类型题目转换为图论。
    1. 模拟
    1. 数学推导
    1. 字符串：匹配算法、正则匹配、通配符匹配、转为图算法
    1. 滑动窗口
    1. 哈希
    1. 高级数据结构
1. 一些建议：
    1. 从时间空间约束上寻找提示
    1. 从暴力法入手，观察搜索树结构，逐渐优化
# 题型分类
- :green_circle: 简单
- :yellow_circle: 中等
- :red_circle: 困难

## 数学
1. :green_circl: 914：卡牌分组。简单的gcd（最大公约数）题目。[欧几里得gcd、扩展欧几里得](https://zhuanlan.zhihu.com/p/58241990)

## 二分
1. :yellow_circle: 33：[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)。自己写了很久总有问题。[精选题解](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/solution/ji-jian-solution-by-lukelee/)。很考虑分类讨论能力和对二分查找的理解。**值得二刷**。

## 字符串
1. :green_circle: 28：[实现strstr](https://leetcode-cn.com/problems/implement-strstr/)。平均意义上又好写又快的[Sunday算法](https://blog.csdn.net/q547550831/article/details/51860017)。其实就是两条策略：失配后对比主串中此次参加匹配的子串后的下一个字符，如果在模式串中没有，则大跳，否则对齐到模式串从右数第一次出现的位置。
1. :green_circle: 1071：[字符串的最大公因子](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/)。有个**题解非常精妙**，比较str1+str2==str2+str1，如果相等，返回gcd(str1.size(),str2.size())，否则返回0。
1. :yellow_circle: 8：[字符串转整数atoi](https://leetcode-cn.com/problems/string-to-integer-atoi)。凡是计算，都要考虑中间步骤、最终结果，**是否会溢出**。尤其是结果不会溢出，但是粗暴的中间计算会，一定要注意。

## 动态规划
1. :green_circle: 121、122：买卖股票的最佳时机[I](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)/[II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)。有通用DP模板。
1. :yellow_circle: 877：[石子游戏](https://leetcode-cn.com/problems/stone-game)。博弈类问题的DP。2人博弈通用DP。但实际上，本题先手必胜。

## 哈希表
1. :green_circle: 219：[存在重复元素II](https://leetcode-cn.com/problems/contains-duplicate-ii/)。滑动窗口+哈希表。

## 多指针
1. :green_circle: 26：[删除数组重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)。输入已经有序，单向双指针扫描。
1. :green_circle: 141：[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)。快慢指针。如果链表中存在循环，一定会相遇。
1. :green_circle: 27：[移除元素](https://leetcode-cn.com/problems/remove-element/)。从左右两侧分别开始，将需要移除的元素覆盖。
1. :yellow_circle: 3：[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)。
1. :yellow_circle: 15：[三数之和](https://leetcode-cn.com/problems/3sum)。排序加对撞指针（双指针相向运动，一般只有输入有序时这样用）。**值得二刷**
1. :yellow_circle: 16：[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest)。依然是排序加对撞。
1. 

## 分治
1. :green_circle: 70：[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)。经典斐波那契问题。矩阵快速幂。

## 位运算
1. :green_circle: 231：[2的幂](https://leetcode-cn.com/problems/power-of-two/solution/)。直接用n&n-1。

## 并发
1. :green_circle: 1114：[按序打印](https://leetcode-cn.com/problems/print-in-order/solution/)。条件变量、锁。

## 数据结构
1. :red_circle: 5999：[统计数组中好三元组数目](https://leetcode-cn.com/problems/count-good-triplets-in-an-array/)。核心题解思路是，以变量y遍历第一个列表，视作三元组的中间元素，并统计第二个列表中，位于y前面的变量中，有多少也在第一个列表中y位置之前出现过。这个统计的信息，恰好可以用[树状数组](https://zhuanlan.zhihu.com/p/93795692)来进行维护。由此达到$O(nlogn)$。
1. :green_circle: 496：[下一个更大元素I](https://leetcode-cn.com/problems/next-greater-element-i/)。自己写的是$O(n^2)$，题解中推荐使用单调栈。

## 待整理算法博客链接
1. [后缀树详解及其具体应用](https://blog.csdn.net/Yuzhiyuxia/article/details/24305683)、[后缀树Ukkonen构造法](https://blog.csdn.net/smbroe/article/details/42362347)、[后缀树系列一:概念以及Ukk实现原理](https://blog.csdn.net/fjsd155/article/details/80211145)
1. [后缀数组解析及应用](https://blog.csdn.net/yxuanwkeith/article/details/50636898?_=_)
1. [二叉树最近公共祖先（LCA）详解](https://www.hrwhisper.me/algorithm-lowest-common-ancestor-of-a-binary-tree/)
1. [July《十五个经典算法研究与总结》索引](https://blog.csdn.net/sailinglt/article/details/79323509)
1. 字符串匹配[BF、KMP、BM、Sunday详解](https://www.cnblogs.com/Syhawk/p/4077295.html)、[从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)、[BF、RK、BM、KMP、Trie树、AC自动机](https://blog.csdn.net/weixin_40805537/article/details/89044710)
1. [SGI版本STL源码中的hashtable（上）](https://blog.csdn.net/Move_now/article/details/78022963)
1. [大数乘法高效算法](https://blog.csdn.net/u010983881/article/details/77503519)
1. [线段树详解](https://www.cnblogs.com/AC-King/p/7789013.html)

## 其他
1. :green_circle: 665：[非递减数列](https://leetcode-cn.com/problems/non-decreasing-array/)。不简单的简单题，**值得二刷**。考虑情况要完整。
1. :yellow_circle: 78：[求全部子集](https://leetcode-cn.com/problems/subsets)。使用vector一定减少内存重分配。

# 经典书籍
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
# 语言相关要点
1. C++：
    1. IEEE754数字标准在C++的实现：[传送门](https://zh.cppreference.com/w/cpp/language/types)

# 一些优化思路
1. 用计算代替分支：分支跳转的代价较高。
1. 尽量将变量定义放在循环外。
1. 多使用位运算。
    - 除2，等价于右移>>1
    - 乘2，等价于左移<<1
    - %2，等价于&1


# 参考
1. 《编程之法：面试和算法心得》：尤其里面的海量数据处理一章值得一读。
1. 《算法导论》：经典中的基础
1. [知乎：有哪些学习算法的网站推荐](https://www.zhihu.com/question/20368410/answer/954418248)