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
- 颜色含义
    - :green_circle: 简单
    - :yellow_circle: 中等
    - :red_circle: 困难
- 注
    - 篇幅所限，仅记录较有代表性的题目

## 模拟
1. :yellow_circle: 950：[按递增顺序显示卡牌](https://leetcode-cn.com/problems/reveal-cards-in-increasing-order/)。重新排序卡牌，使得满足一定条件。考研模拟的思路。

## 搜索
1. :yellow_circle: 39：[组合总数](https://leetcode-cn.com/problems/combination-sum/)。标准DFS。
1. :yellow_circle: 40：[组合总数II](https://leetcode-cn.com/problems/combination-sum-ii/)。39题的变种。需要使用一些剪枝来避免重复。
1. :yellow_circle: 94：[二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)。普通写法没什么。迭代写法就需要一点东西了，可参考[Morris遍历](https://www.freesion.com/article/2475651348/)。另一篇[更通用的写法](https://blog.csdn.net/softwarex4/article/details/95986102)参考。
1. :yellow_circle: 22：[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)。虽然是搜索题，但是想要写一个高效的剪枝版并不是很容易。
1. :yellow_circle: 133：[克隆图](https://leetcode-cn.com/problems/clone-graph/)。BFS，需要注意的是图的各种特殊情况，自环，重复边等。
1. :yellow_circle: 491：[递增子序列](https://leetcode-cn.com/problems/increasing-subsequences/)。DFS+少量优化。通过添加当前层使用的数字的记录来避免生成重复的记录。**值得二刷**
1. :yellow_circle: 638：[大礼包](https://leetcode-cn.com/problems/shopping-offers/)。经典的记忆化DFS。开辟一个类似于状态数组，记录指定状态下的最优解，避免重复搜索。**值得二刷**。
1. :yellow_circle: 863：[二叉树中所有距离为K的节点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)。经典的多次搜索的题目，[题解](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/solution/er-cha-shu-zhong-suo-you-ju-chi-wei-k-de-qbla/)把问题分段处理。
1. :yellow_circle: 417：[太平洋大西洋水流问题](https://leetcode-cn.com/problems/pacific-atlantic-water-flow/)。这一款[题解](https://leetcode-cn.com/problems/pacific-atlantic-water-flow/solution/shui-wang-gao-chu-liu-by-xiaohu9527-xxsx/)是经典的变换问题，逆向DFS。地图类搜索题目的搜索起点、终点都可以尝试进行对调。有时能降低复杂度或实现难度。

## 数学
1. :green_circle: 914：[卡牌分组](https://leetcode-cn.com/problems/x-of-a-kind-in-a-deck-of-cards)。简单的gcd（最大公约数）题目。[欧几里得gcd、扩展欧几里得](https://zhuanlan.zhihu.com/p/58241990)
1. :yellow_circle: 343：[整数拆分](https://leetcode-cn.com/problems/integer-break/)。从数字分析入手，发现所有的拆分方式中，拆出最多的3是最优解。
1. :yellow_circle: 365：[水壶问题](https://leetcode-cn.com/problems/water-and-jug-problem/)。基础款可以写个BFS。但由于是判定问题，其实可以直接参考**扩展欧几里得**、**裴蜀定理**。

## 二分
1. :yellow_circle: 33：[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)。自己写了很久总有问题。[精选题解](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/solution/ji-jian-solution-by-lukelee/)。很考虑分类讨论能力和对二分查找的理解。**值得二刷**。
1. :yellow_circle: 540：[有序数组中的单一元素](https://leetcode-cn.com/problems/single-element-in-a-sorted-array)。利用有序性。

## 字符串
1. :green_circle: 28：[实现strstr](https://leetcode-cn.com/problems/implement-strstr/)。平均意义上又好写又快的[Sunday算法](https://blog.csdn.net/q547550831/article/details/51860017)。其实就是两条策略：失配后对比主串中此次参加匹配的子串后的下一个字符，如果在模式串中没有，则大跳，否则对齐到模式串从右数第一次出现的位置。
1. :green_circle: 1071：[字符串的最大公因子](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/)。有个**题解非常精妙**，比较str1+str2==str2+str1，如果相等，返回gcd(str1.size(),str2.size())，否则返回0。
1. :yellow_circle: 8：[字符串转整数atoi](https://leetcode-cn.com/problems/string-to-integer-atoi)。凡是计算，都要考虑中间步骤、最终结果，**是否会溢出**。尤其是结果不会溢出，但是粗暴的中间计算会，一定要注意。
1. :构建回文串检测: 1177：[构建回文串检测](https://leetcode-cn.com/problems/can-make-palindrome-from-substring/)。计算各个奇偶性并进行判断即可。
1. :yellow_circle: 820：[单词的压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/)。使用Trie树(前缀树、字典树)[题解](https://leetcode-cn.com/problems/short-encoding-of-words/solution/dan-ci-de-ya-suo-bian-ma-by-leetcode-solution/)。
1. :yellow_circle: 151：[反转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)。小技巧，先翻转小单词，再翻转整体，等价于把单词位置进行对调。

## 动态规划
1. :green_circle: 121、122：买卖股票的最佳时机[I](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)/[II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)。有通用DP模板。
1. :yellow_circle: 877：[石子游戏](https://leetcode-cn.com/problems/stone-game)。博弈类问题的DP。但实际上，本题先手必胜。**值得二刷**。
1. :yellow_circle: 241：[为运算表达式设计优先级](https://leetcode-cn.com/problems/different-ways-to-add-parentheses/)。感觉可以dp。**好像没自己写？**
1. :yellow_circle: 300：[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)。$O(n^2)$的好写。思考一下$O(nlogn)$的。[题解](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-by-leetcode-soluti/)精妙之处在于状态的设计。dp的优化方向之一，让状态具备单调性。
1. :yellow_circle: 96：[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)。dp就可以。也了解一下[卡塔兰数](https://leetcode-cn.com/problems/unique-binary-search-trees/solution/bu-tong-de-er-cha-sou-suo-shu-by-leetcode-solution/)。
1. :yellow_circle: 95：[不同的二叉搜索树II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)。本题需要直接返回所有的树，需要快速且不重复的生成。[题解](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-2-7/)中提到了树的同构问题。**值得二刷**
1. :yellow_circle: 918：[环形子数组的最大和](https://leetcode-cn.com/problems/maximum-sum-circular-subarray/)。注意子数组（必连续）和子序列（不必连续）的区别。[题解](https://leetcode-cn.com/problems/maximum-sum-circular-subarray/solution/huan-xing-zi-shu-zu-de-zui-da-he-by-leetcode/)为kadane算法及其变种。**值得二刷**。
1. :yellow_circle: 337：[打家劫舍III](https://leetcode-cn.com/problems/house-robber-iii/)。不太常见的树形DP。$curNode=f(leftSon,leftSonSon,rightSon,rightSonSon)$。
1. :yellow_circle: 494：[目标和](https://leetcode-cn.com/problems/target-sum/)。[题解](https://leetcode-cn.com/problems/target-sum/solution/mu-biao-he-by-leetcode-solution-o0cp/)经过一定的数学变换，变为0/1背包。$\sum(A)+\sum(B)=\sum(Nums), \sum(A)-\sum(B)=S, \sum(A)=(\sum(Nums)+S)/2$。如此目标就变为了求容量为$(\sum(Nums)+S)/2$的背包中集合$a$的问题。
1. :yellow_circle: 221：[最大正方形](https://leetcode-cn.com/problems/maximal-square/)。二维问题（及以上）设计状态方程的经典例子。[题解](https://leetcode-cn.com/problems/maximal-square/solution/zui-da-zheng-fang-xing-by-leetcode-solution/)按照右下角坐标设置状态。$dp[i][j]$代表右下角坐标为$(i,j)$的矩形的最大面积。则状态转移类似$dp[i][j]=f(dp[i-1][j-1],dp[i][j-1],dp[i-1][j])$

## 哈希表
1. :green_circle: 219：[存在重复元素II](https://leetcode-cn.com/problems/contains-duplicate-ii/)。滑动窗口+哈希表。
1. :yellow_circle: 945：[使数组唯一的最小增量值](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/)。把哈希表的线性探测思路应用到算法题目中。[题解](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/solution/ji-shu-onxian-xing-tan-ce-fa-onpai-xu-onlogn-yi-ya/)还结合了路径压缩。

## 多指针 & 滑动窗口
1. :green_circle: 26：[删除数组重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)。输入已经有序，单向双指针扫描。
1. :green_circle: 141：[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)。快慢指针。如果链表中存在循环，一定会相遇。
1. :green_circle: 27：[移除元素](https://leetcode-cn.com/problems/remove-element/)。从左右两侧分别开始，将需要移除的元素覆盖。
1. :yellow_circle: 3：[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)。
1. :yellow_circle: 15：[三数之和](https://leetcode-cn.com/problems/3sum)。排序加对撞指针（双指针相向运动，一般只有输入有序时这样用）。**值得二刷**
1. :yellow_circle: 16：[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest)。依然是排序加对撞。
1. :yellow_circle: 238：[除自身外的乘积](https://leetcode-cn.com/problems/product-of-array-except-self)。有的时候可以从复杂度要求思考方法。
1. :yellow_circle: 18：[四数之和](https://leetcode-cn.com/problems/4sum)。**值得二刷**。
1. :yellow_circle: 1151：[最少交换次数来组合所有的1](https://leetcode-cn.com/problems/minimum-swaps-to-group-all-1s-together/)。[非会员链接](https://blog.csdn.net/qq_17550379/article/details/99169544)。滑动窗口+前缀和。滑动窗口的关键之一就是如何设置窗口大小。

## 分治
1. :green_circle: 70：[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)。经典斐波那契问题。矩阵快速幂。
1. :yellow_circle: 215：[数组中第K个最大的元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)。可以使用[BFPRT算法](https://www.lyclife.com/2021/07/%E7%AE%97%E6%B3%95%E5%AF%BC%E8%AE%BA%E5%85%B6%E5%9B%9B%E4%B8%AD%E4%BD%8D%E6%95%B0%E5%92%8C%E9%A1%BA%E5%BA%8F%E7%BB%9F%E8%AE%A1/)，理论上更快，但是其实常数很高，也可以用小顶堆来实现，理论上复杂度高一些。
1. :yellow_circle: 148：[排序链表](https://leetcode-cn.com/problems/sort-list/)。进阶要求是$O(nlogn)$时间和$O(1)$的空间。只能使用自底向上的原地归并排序。

## 排序
1. :yellow_circle: 220：[存在重复元素III](https://leetcode-cn.com/problems/contains-duplicate-iii/)。使用滑动窗口+平衡树的方式可以很容易得到$O(nlogk)$的时间复杂度。但[题解](https://leetcode-cn.com/problems/contains-duplicate-iii/solution/cun-zai-zhong-fu-yuan-su-iii-by-leetcode-bbkt/)使用桶排序的思想来解决**abs值是否在一个区间内**的判断问题，需要判断当前桶和相邻桶。
1. :yellow_circle: 355：[设计推特](https://leetcode-cn.com/problems/design-twitter/)。思考的时候可以参考下基础算法题。本体实际上是合并k个有序链表。

## 位运算
1. :green_circle: 231：[2的幂](https://leetcode-cn.com/problems/power-of-two/solution/)。直接用n&n-1。
1. :yellow_circle: 201：[数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)。纯暴力遍历不可取。题解即求两个数字的[二进制下的公共前缀](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)，里面提到了用于去除二进制串最右侧1的，Brian Kernighan算法。

## 并发
1. :green_circle: 1114：[按序打印](https://leetcode-cn.com/problems/print-in-order/solution/)。条件变量、锁。

## 图论
1. :yellow_circle: 721：[账户合并](https://leetcode-cn.com/problems/accounts-merge/)。基础并查集题目。
1. :yellow_circle: 1162：[地图分析](https://leetcode-cn.com/problems/as-far-from-land-as-possible/)。很有代表性的一道题目，巧妙地把问题转换为多源点BFS。[题解](https://leetcode-cn.com/problems/as-far-from-land-as-possible/solution/di-tu-fen-xi-by-leetcode-solution/)中甚至还有更精妙的DP解法。**值得二刷**。
1. :yellow_circle: 207：[课程表](https://leetcode-cn.com/problems/course-schedule/)。拓扑排序。[wiki](http://en.wikipedia.org/wiki/Topological_sorting)。
1. :red_circle: 332：[重新安排行程](https://leetcode-cn.com/problems/reconstruct-itinerary/)。学习图论概念：[欧拉图&半欧拉图&欧拉回路&半欧通路](https://oi-wiki.org/graph/euler/)。[Hierholzer算法题解](https://leetcode-cn.com/problems/reconstruct-itinerary/solution/zhong-xin-an-pai-xing-cheng-by-leetcode-solution/)。**值得二刷**。
1. :yellow_circle: 310：[最小高度树](https://leetcode-cn.com/problems/minimum-height-trees/)。自己写的是普通的BFS。[题解]()中揭示了本题实际上是逐步删除度为1的节点，即拓扑排序。或者说这是一类，两端烧香求中点的题目。
1. :yellow_circle: 743：[网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)。基础款单源有向最短路。邻接表+djikstra+堆优化。

## 数据结构
1. :red_circle: 2179：[统计数组中好三元组数目](https://leetcode-cn.com/problems/count-good-triplets-in-an-array/)。核心题解思路是，以变量y遍历第一个列表，视作三元组的中间元素，并统计第二个列表中，位于y前面的变量中，有多少也在第一个列表中y位置之前出现过。这个统计的信息，恰好可以用[树状数组](https://zhuanlan.zhihu.com/p/93795692)来进行维护。由此达到$O(nlogn)$。
1. :green_circle: 496：[下一个更大元素I](https://leetcode-cn.com/problems/next-greater-element-i/)。自己写的是$O(n^2)$，题解中推荐使用单调栈。
1. :yellow_circle: 731：[我的日程安排]。看了题解，第一种是可以用两个set，分别存储无重叠时间段，和有一重重叠的时间段，新加入时间段不允许和一重重叠的时间段再重叠。另有题解[边界计数](https://leetcode-cn.com/problems/my-calendar-ii/solution/wo-de-ri-cheng-an-pai-biao-ii-by-leetcode/)，[线段树基础题型](https://leetcode-cn.com/problems/my-calendar-ii/solution/xian-duan-shu-dong-tai-kai-dian-lan-duo-336be/)，**值得二刷**。
1. :yellow_circle: 654：[最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)。单调栈。
1. :yellow_circle: 1395：[统计作战单位数](https://leetcode-cn.com/problems/count-number-of-teams/)。暴力枚举中间值能过。学习以下[题解](https://leetcode-cn.com/problems/count-number-of-teams/solution/tong-ji-zuo-zhan-dan-wei-shu-by-leetcode-solution/)使用离散化+树状数组高效解决。这里用的离散化方法是通过排序，确定其序号作为关键字。
1. :yellow_circle: 109：[从有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)。锻炼基础能力，熟悉前中后序遍历。**值得二刷**。

## 其他
1. :green_circle: 665：[非递减数列](https://leetcode-cn.com/problems/non-decreasing-array/)。不简单的简单题，**值得二刷**。考虑情况要完整。
1. :yellow_circle: 78：[求全部子集](https://leetcode-cn.com/problems/subsets)。使用vector一定减少内存重分配。
1. :yellow_circle: 169：[找众数](https://leetcode-cn.com/problems/majority-element)。Boyer-Moore投票算法。维护当前众数和计数器。相等+1，否则-1。变为0则更换数字。
1. :yellow_circle: 287：[寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)。要求$O(n)$复杂度。题解使用等价变换+[Floyd判圈法](https://www.jianshu.com/p/36a89d938440)，**值得二刷**。
1. :yellow_circle: 138：[复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)。题解方法非常有趣，原地将链表复制一遍，变为a->a'->b->b'->...->nil。然后再对random域赋值。这种原地操作的思路值得思考。

# 待整理算法博客链接
1. [后缀树详解及其具体应用](https://blog.csdn.net/Yuzhiyuxia/article/details/24305683)、[后缀树Ukkonen构造法](https://blog.csdn.net/smbroe/article/details/42362347)、[后缀树系列一:概念以及Ukk实现原理](https://blog.csdn.net/fjsd155/article/details/80211145)
1. [后缀数组解析及应用](https://blog.csdn.net/yxuanwkeith/article/details/50636898?_=_)
1. [二叉树最近公共祖先（LCA）详解](https://www.hrwhisper.me/algorithm-lowest-common-ancestor-of-a-binary-tree/)
1. [July《十五个经典算法研究与总结》索引](https://blog.csdn.net/sailinglt/article/details/79323509)
1. 字符串匹配[BF、KMP、BM、Sunday详解](https://www.cnblogs.com/Syhawk/p/4077295.html)、[从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)、[BF、RK、BM、KMP、Trie树、AC自动机](https://blog.csdn.net/weixin_40805537/article/details/89044710)
1. [SGI版本STL源码中的hashtable（上）](https://blog.csdn.net/Move_now/article/details/78022963)
1. [大数乘法高效算法](https://blog.csdn.net/u010983881/article/details/77503519)
1. [线段树详解](https://www.cnblogs.com/AC-King/p/7789013.html)

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