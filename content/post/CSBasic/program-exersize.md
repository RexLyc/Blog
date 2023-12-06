---
title: "编程题笔记：题目记录"
date: 2021-12-30T16:29:39+08:00
categories:
- 求职
- 习题
tags:
- 滚动更新
- 习题
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algo-exercises.jpg
math: true
---
题目内容来自leetcode、牛客等平台，偶有面试笔试真题。
<!--more-->
## 总结
1. 未特殊说明则惯用语言为C++，偶尔会使用Java、Python，这些情况会特别标出。
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
    1. 图算法：并查集、拓扑排序、欧拉图、最短路、网络流、哈密顿路径、欧拉路径
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
## 题型分类
- 颜色含义
    - :green_circle: 简单
    - :yellow_circle: 中等
    - :red_circle: 困难
- 注
    - 篇幅所限，仅记录较有代表性的题目

###  模拟
1. :yellow_circle: 950：[按递增顺序显示卡牌](https://leetcode-cn.com/problems/reveal-cards-in-increasing-order/)。重新排序卡牌，使得满足一定条件。考验模拟的思路。

###  搜索
1. :yellow_circle: 2477：[到达首都的最小油耗](https://leetcode.cn/problems/minimum-fuel-cost-to-report-to-the-capital/description/)。自己写了个类似拓扑排序。实际上夸张了，这题直接从根开始深搜就行。每个节点统计子树的全部乘客，再向上返回。
1. :rec_circle: 2258: [逃离火灾](https://leetcode.cn/problems/escape-the-spreading-fire/description/)。唯一需要特殊考虑的情况，是只有安全屋格子允许人和火同时抵达。优化思路，从起始位置和着火点分别BFS$\to$从安全屋反向BFS并记录路径$\to$不需要记录路径，只需要标记路径是从左侧还是右侧。题解里还有对等待时间二分查找+BFS，虽然效率低，但是二分的思路很有代表性。
1. :yellow_circle: 39：[组合总数](https://leetcode-cn.com/problems/combination-sum/)。标准DFS。
1. :yellow_circle: 40：[组合总数II](https://leetcode-cn.com/problems/combination-sum-ii/)。39题的变种。需要使用一些剪枝来避免重复。
1. :yellow_circle: 94：[二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)。普通写法没什么。迭代写法就需要一点东西了，可参考[二叉树遍历方法大全](https://www.cnblogs.com/sunshuyi/p/12680577.html)、[Morris遍历](https://www.freesion.com/article/2475651348/)。尤其注意思考Morris后序遍历的反转操作。
1. :yellow_circle: 22：[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)。虽然是搜索题，但是想要写一个高效的剪枝版并不是很容易。
1. :yellow_circle: 133：[克隆图](https://leetcode-cn.com/problems/clone-graph/)。BFS，需要注意的是图的各种特殊情况，自环，重复边等。
1. :yellow_circle: 491：[递增子序列](https://leetcode-cn.com/problems/increasing-subsequences/)。DFS+少量优化。通过添加当前层使用的数字的记录来避免生成重复的记录。**值得二刷**
1. :yellow_circle: 638：[大礼包](https://leetcode-cn.com/problems/shopping-offers/)。经典的记忆化DFS。开辟一个类似于状态数组，记录指定状态下的最优解，避免重复搜索。**值得二刷**。
1. :yellow_circle: 863：[二叉树中所有距离为K的节点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)。经典的多次搜索的题目，[题解](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/solution/er-cha-shu-zhong-suo-you-ju-chi-wei-k-de-qbla/)把问题分段处理。
1. :yellow_circle: 417：[太平洋大西洋水流问题](https://leetcode-cn.com/problems/pacific-atlantic-water-flow/)。这一款[题解](https://leetcode-cn.com/problems/pacific-atlantic-water-flow/solution/shui-wang-gao-chu-liu-by-xiaohu9527-xxsx/)是经典的变换问题，逆向DFS。地图类搜索题目的搜索起点、终点都可以尝试进行对调。有时能降低复杂度或实现难度。
1. :red_circle: 127：[单词接龙](https://leetcode-cn.com/problems/word-ladder/)。经典双向BFS优化。
1. :yellow_circle: 254：[因子的组合](https://leetcode-cn.com/problems/factor-combinations/)。[非会员链接](https://www.cnblogs.com/grandyang/p/5332722.html)。搜索，但是搜索的因子保持单调，以避免重复。
1. :yellow_circle: 351：[安卓系统手势解锁](https://leetcode-cn.com/problems/android-unlock-patterns)。**暴力生成并判别**是否满足要求即可。不用费力的进行正确生成。
1. :yellow_circle: 465：[我能赢吗](https://leetcode-cn.com/problems/can-i-win/)。不可多得的博弈类型DFS，这类题目基本的思路就是记忆化的DFS。本题DFS的返回值代表是否有必胜策略。可以看看[题解](https://leetcode-cn.com/problems/can-i-win/solution/464-wo-neng-ying-ma-dai-bei-wang-lu-de-d-qu1t/)。
1. :yellow_circle: 1530：[好叶子节点对的数量](https://leetcode-cn.com/problems/number-of-good-leaf-nodes-pairs/)。DFS时选择一个好的返回值真的很重要。本题可以考虑返回当前节点各深度的子孙节点列表。
1. :red_circle: 124：[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)。经典的对树结构的搜索。通用思路就是，分别递归搜索左子树、右子树，处理二者返回值并返回。
1. :red_circle: 301：[删除无效的括号](https://leetcode-cn.com/problems/remove-invalid-parentheses/)。DFS剪枝典范。对每个括号进行是否删除的搜索，但可以结合当前左右括号数量、总的左右括号数量进行一定程度的剪枝。
1. :red_circle: 1036：[逃离大迷宫](https://leetcode-cn.com/problems/escape-a-large-maze/)。BFS略加优化能过，但[题解](https://leetcode-cn.com/problems/escape-a-large-maze/)效率更高。搜索类题目，先看输入数据规模，选择合理的搜索内容。本题就是，不是直接搜索通路，而是判断障碍节点能否单独包围住起始点或终止点。
1. :red_circle: 1368：[使网格图至少有一条有效路径的最小代价](https://leetcode-cn.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/)。可以用最短路。但本题更有价值的思路是0-1广度优先搜索。原理很简单，BFS的队列使用双端队列，权为0的边push_front，1的push_back。
1. :red_circle: 980：[不同路径III](https://leetcode-cn.com/problems/unique-paths-iii/)。记忆化搜索。通过保存$dp(current_point,unseen_points)$来减少重复搜索。其中unseen_points代表了尚未遍历的节点列表。
1. :red_circle: 407：[接雨水II](https://leetcode-cn.com/problems/trapping-rain-water-ii/)。直观上是寻找低点并扩散，但实际本题是从外向内扩展，即假定外围有一圈水，并且从最矮处开始扩展（短板效应）。经典的**反向思路**的BFS。BFS可以偶尔尝试把从内向外，转为从外向内。

###  数学
1. :green_circle: 914：[卡牌分组](https://leetcode-cn.com/problems/x-of-a-kind-in-a-deck-of-cards)。简单的gcd（最大公约数）题目。[欧几里得gcd、扩展欧几里得](https://zhuanlan.zhihu.com/p/58241990)
1. :yellow_circle: 343：[整数拆分](https://leetcode-cn.com/problems/integer-break/)。从数字分析入手，发现所有的拆分方式中，拆出最多的3是最优解。
1. :yellow_circle: 365：[水壶问题](https://leetcode-cn.com/problems/water-and-jug-problem/)。基础款可以写个BFS。但由于是判定问题，其实可以直接参考**扩展欧几里得**、**裴蜀定理**。
1. :yellow_circle: 279：[完全平方数](https://leetcode-cn.com/problems/perfect-squares/)。可以按照无穷背包进行处理。但[题解](https://leetcode-cn.com/problems/perfect-squares/solution/wan-quan-ping-fang-shu-by-leetcode-solut-t99c/)提供了一个数学解法。四平方和定理。只需要记住一点，若数字可表示为$4^k(8m+7)$，其中$k,m \in \mathbb{N}$，则必定只能用4个平方数求和，否则可以用至多3个平方数求和得出。
1. :red_circle: 891：[子序列宽度之和](https://leetcode-cn.com/problems/sum-of-subsequence-widths/)。逆向思维，寻找子序列中的max/min$\to$固定max/min，计算有多少个序列。固定max/min，需要先对数组排序，然后按顺序取。随后可以通过数学推理，计算出所有序列的和的一个解析解表达式。
1. :yellow_circle: 2028：[找出缺失的观测数据](https://leetcode-cn.com/problems/find-missing-observations/)。可以写搜索，效率还行（每种数字的数量都有上下限可用）。但[题解](https://leetcode-cn.com/problems/find-missing-observations/solution/zhao-chu-que-shi-de-guan-ce-shu-ju-by-le-0z7j/)给出了一种已知均值和整数的总数，可以直接构造出整数数组的方法。
1. :red_circle: 1259：[不相交的握手](https://leetcode-cn.com/problems/handshakes-that-dont-cross/)。[非会员传送门](https://www.acwing.com/file_system/file/content/whole/index/content/1528105/)。可以用DP，但用数学更好。卡塔兰数+Lucas定理。
2. :yellow_circle: 1094：[拼车](https://leetcode.cn/problems/car-pooling/description/)。看题解，用**差分数组**思想，前缀和和差分数组互为逆运算。将上下车视作加减人数，则可把任意位置的乘客数量看作对这个加减法的前缀和，最笨的方式是计算所有位置的前缀和。但实际上本体是一个区间修改，而且区间很小，那么可以做前缀和的逆运算，差分数组，差分数组只需要在区间修改的起始和终止位置进行调整。最后再用差分数组构建前缀和即可。

###  二分
> 二分查找，最好的表示方式就是，left边界，remain查找区间长度。注意left+remain就是第一个超出区间的值，但是也有可能是查找结果，参考vector容器的end()。参考C++标准库[lower_bound的可能实现](https://en.cppreference.com/w/cpp/algorithm/lower_bound)。
 
1. :yellow_circle: 33：[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)。自己写了很久总有问题。[精选题解](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/solution/ji-jian-solution-by-lukelee/)。很考虑分类讨论能力和对二分查找的理解。**值得二刷**。
1. :yellow_circle: 540：[有序数组中的单一元素](https://leetcode-cn.com/problems/single-element-in-a-sorted-array)。利用有序性。
1. :red_circle: 4：[寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)。经典的二分查找题目。普通的二分$O(log(m+n))$，以及利用中位数性质的二分$O(log(min(m,n))$（典型的**数量关系**，一个数组划分后，另一个数组理论上的划分位置是固定的）。还要注意特殊情况处理。**值得二刷**。
1. :red_circle: 719：[找出第k小的距离对](https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance)。看的[题解](https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance/solution/hei-ming-dan-zhong-de-sui-ji-shu-by-leetcode/)：二分查找 + **非常妙**的双指针。一般来说，计算全部的距离（本题的求解目标值）的复杂度，是要高于搜索合适的距离值（尤其是使用二分搜索）。可思考求第K大这个经典问题。
1. :yellow_circle: 57：[插入区间](https://leetcode-cn.com/problems/insert-interval/)。C++下本题最快捷的办法就是自己写一个lower_bound、upper_bound所需要的Compare函数。
1. :red_circle: 1095：[山脉数组中查找目标值](https://leetcode-cn.com/problems/find-in-mountain-array/)。还是区间二分，需要先从数组中查找到最大值。很有新意的一道题目，考验对于各种情况的分析。[题解](https://leetcode-cn.com/problems/find-in-mountain-array/solution/shan-mai-shu-zu-zhong-cha-zhao-mu-biao-zhi-by-leet/)非常简洁，**值得二刷**。
1. :red_circle: 363：[矩形区域不超过K的最大数值和](https://leetcode-cn.com/problems/max-sum-of-rectangle-no-larger-than-k/)。本题是前缀和+二分。注意当题目中出现**不超过XXX**的字样的时候，说明出现了不等式，而这一般意味着可以对中间结果进行排序（或使用动态有序的数据结构），并用二分进行查找。
1. :red_circle: 410：[分割数组的最大值](https://leetcode-cn.com/problems/split-array-largest-sum/)。DP和记忆化搜索复杂度太高怎么办？一定要学[题解](https://leetcode-cn.com/problems/split-array-largest-sum/solution/fen-ge-shu-zu-de-zui-da-zhi-by-leetcode-solution/)，非常经典的二分思路，对待求结果进行**二分测试**。先设定一个最大值，超过该值就进行划分，如果划分数组的数量符合要求。再进一步用二分确定最大值的范围。
    - 仔细思考。如果搜索，则相当于是求出了过多的信息（很多种划分方式的最大值都被算出来了）。
    - 只要是结果有大小顺序，都可以用二分测试来计算。
1. :yellow_circle: 2560: [打家劫舍 IV](https://leetcode.cn/problems/house-robber-iv/description/)。经典最小化最大值问题。这类问题就是针对这个值进行二分。而不是真的去模拟求出所有的最大值。另外该题需要想明白两点：先选一定不比后选更差（贪心策略的正确性，类似于优先选择最早完成的任务），另一点是二分的最终结果一定会存在于数组中（不用担心二分的结果不在原数组）。

###  字符串
1. :red_circle: 828：[统计字符串中唯一字符](https://leetcode.cn/problems/count-unique-characters-of-all-substrings-of-a-given-string/description)。和题解的思路是一致的。把计算子串中的唯一字符，转向计算一个字符能在多少个子串中唯一。但是计算这种子串卡了很久。注意理解：计算在$(begin,end)$开区间内，包含middle位置的字符的子串的数量（其中$begin<middle<end$），就是$(middle-begin)*(end-middle)$。
1. :red_circle: 1044：[最长重复子串](https://leetcode.cn/problems/longest-duplicate-substring/description/)。最长重复子串，等价于求所有后缀的最长公共前缀。后缀数组的样板题目。
2. :green_circle: 28：[实现strstr](https://leetcode-cn.com/problems/implement-strstr/)。平均意义上又好写又快的[Sunday算法](https://blog.csdn.net/q547550831/article/details/51860017)。其实就是两条策略：失配后对比主串中此次参加匹配的子串后的下一个字符，如果在模式串中没有，则大跳，否则对齐到模式串从右数第一次出现的位置。
3. :green_circle: 1071：[字符串的最大公因子](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/)。有个**题解非常精妙**，比较str1+str2==str2+str1，如果相等，返回gcd(str1.size(),str2.size())，否则返回0。
4. :yellow_circle: 8：[字符串转整数atoi](https://leetcode-cn.com/problems/string-to-integer-atoi)。凡是计算，都要考虑中间步骤、最终结果，**是否会溢出**。尤其是结果不会溢出，但是粗暴的中间计算会，一定要注意。
5. :构建回文串检测: 1177：[构建回文串检测](https://leetcode-cn.com/problems/can-make-palindrome-from-substring/)。计算各个奇偶性并进行判断即可。
6. :yellow_circle: 820：[单词的压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/)。使用Trie树(前缀树、字典树)[题解](https://leetcode-cn.com/problems/short-encoding-of-words/solution/dan-ci-de-ya-suo-bian-ma-by-leetcode-solution/)。
7. :yellow_circle: 151：[反转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)。小技巧，先翻转小单词，再翻转整体，等价于把单词位置进行对调。
8. :yellow_circle: 647：[回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)。先使用[Manacher算法](https://www.jianshu.com/p/116aa58b7d81)求出全部回文子串，结果显然。**值得二刷**。
9. :red_circle: 1044：[最长重复子串](https://leetcode-cn.com/problems/longest-duplicate-substring/)。二分法+Rabin-Karp算法（实现不好就超时）。后缀数组+最长公共子串。***没做完***
10. :red_circle: 65：[有效数字](https://leetcode-cn.com/problems/valid-number/)。直接上状态机，参考[题解](https://leetcode-cn.com/problems/valid-number/solution/biao-qu-dong-fa-by-user8973/)中提到的编译原理DFA。
11. :red_circle: 466：[统计重复个数](https://leetcode-cn.com/problems/count-the-repetitions/)。题材非常新颖的一道。统计字符串之间的循环节。注意：参考小数的循环，字符串的循环节不一定从第一节字符串开始（前面可能有一个前缀）。[题解传送门](https://leetcode-cn.com/problems/count-the-repetitions/solution/tong-ji-zhong-fu-ge-shu-by-leetcode-solution/)。**值得二刷**。
12. :red_circle: 1153：[字符串转化](https://leetcode-cn.com/problems/string-transforms-into-another-string/)。[非会员传送门](https://blog.csdn.net/qq_17550379/article/details/99404963)。很新颖的一道题目。判断类题目不需要给出解，**只需要判断**。本题只需要判断是否可能完成转换：不能具有全部26个字母，不能有相同字母需要修改为不同字母。
13. :red_circle: 212：[单词搜索II](https://leetcode-cn.com/problems/word-search-ii/)。经典前缀树（字典树、Trie树）题目。可以看一下题解的[优化实现](https://leetcode-cn.com/problems/word-search-ii/solution/dan-ci-sou-suo-ii-by-leetcode-solution-7494/)。
14. :red_circle: 6093：[设计一个文本编辑器](https://leetcode.cn/problems/design-a-text-editor/)。（看了题解）脑筋急转弯级别的困难题。很容易陷入到写一个链表结构的数据结构中（很难写）。但实际上，可以看作是类似两个栈，分别是前缀和后缀。左右移动就是两个栈来回压入，插入删除则是对前缀的压入和弹出。

###  动态规划
1. :red_circle: 689：[三个无重叠子数组的最大和](https://leetcode.cn/problems/maximum-sum-of-3-non-overlapping-subarrays/description/)。思考方向比较顺畅：从一个最大和子数组，到两个，再到三个，可以很自然的思考出来动态规划方程。本质还是背包问题。
1. :green_circle: 121、122：买卖股票的最佳时机[I](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)/[II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)。有通用DP模板。
1. :yellow_circle: 309：[最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)。$dp[i][j]$代表第i天结束，手中有j支股票的最大收益。
1. :yellow_circle: 877：[石子游戏](https://leetcode-cn.com/problems/stone-game)。博弈类问题的DP。但实际上，本题先手必胜。**值得二刷**。
1. :yellow_circle: 241：[为运算表达式设计优先级](https://leetcode-cn.com/problems/different-ways-to-add-parentheses/)。感觉可以dp。**好像没自己写？**
1. :yellow_circle: 300：[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)。$O(n^2)$的好写。思考一下$O(nlogn)$的。[题解](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-by-leetcode-soluti/)精妙之处在于状态的设计。dp的优化方向之一，让状态具备单调性。也叫做LIS问题（最长不下降子序列）
1. :yellow_circle: 96：[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)。dp就可以。也了解一下[卡塔兰数](https://leetcode-cn.com/problems/unique-binary-search-trees/solution/bu-tong-de-er-cha-sou-suo-shu-by-leetcode-solution/)。
1. :yellow_circle: 95：[不同的二叉搜索树II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)。本题需要直接返回所有的树，需要快速且不重复的生成。[题解](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-2-7/)中提到了树的同构问题。**值得二刷**
1. :yellow_circle: 918：[环形子数组的最大和](https://leetcode-cn.com/problems/maximum-sum-circular-subarray/)。注意子数组（必连续）和子序列（不必连续）的区别。[题解](https://leetcode-cn.com/problems/maximum-sum-circular-subarray/solution/huan-xing-zi-shu-zu-de-zui-da-he-by-leetcode/)为kadane算法及其变种。**值得二刷**。
1. :yellow_circle: 337：[打家劫舍III](https://leetcode-cn.com/problems/house-robber-iii/)。不太常见的树形DP。$curNode=f(leftSon,leftSonSon,rightSon,rightSonSon)$。
1. :yellow_circle: 494：[目标和](https://leetcode-cn.com/problems/target-sum/)。[题解](https://leetcode-cn.com/problems/target-sum/solution/mu-biao-he-by-leetcode-solution-o0cp/)经过一定的数学变换，变为0/1背包。$\sum(A)+\sum(B)=\sum(Nums), \sum(A)-\sum(B)=S, \sum(A)=(\sum(Nums)+S)/2$。如此目标就变为了求容量为$(\sum(Nums)+S)/2$的背包中集合$a$的问题。
1. :yellow_circle: 221：[最大正方形](https://leetcode-cn.com/problems/maximal-square/)。二维问题（及以上）设计状态方程的经典例子。[题解](https://leetcode-cn.com/problems/maximal-square/solution/zui-da-zheng-fang-xing-by-leetcode-solution/)按照右下角坐标设置状态。$dp[i][j]$代表右下角坐标为$(i,j)$的矩形的最大面积。则状态转移类似$dp[i][j]=f(dp[i-1][j-1],dp[i][j-1],dp[i-1][j])$
1. :yellow_circle: 473：[火柴拼正方形](https://leetcode-cn.com/problems/matchsticks-to-square/)。基本解法是多次DFS。但[题解](https://leetcode-cn.com/problems/matchsticks-to-square/solution/huo-chai-pin-zheng-fang-xing-by-leetcode/)给出了更好的状态压缩的动态规划。即标记每一个火柴是否被使用，作为状态。而且规定任意状态下至多有1个边尚未填满。动态规划的一个典型思路就是，并不需要计算所有状态，只需要证明，所计算的状态链必定包含最优解即可。**值得二刷**。
1. :yellow_circle: 139：[单词拆分](https://leetcode-cn.com/problems/word-break/)。自己写了个回溯+前缀树，经典TLE（Time Limit Exceeded）。其实这道题是个DP，字符串的DP其实挺多的。以$dp[i]$记录目标字符串$s$的子串$substr(0,i)$能否被成功过拆分。状态转移类似背包问题。
1. :red_circle: 140：[单词拆分II](https://leetcode-cn.com/problems/word-break-ii/)。做一定的预处理，获取任意位置分割后的字符串样子，便于使用。
1. :yellow_circle: 131：[分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)。先**预处理**回文位置，再做动态规划。$dp[i]=f(dp[0...i])$。
1. :yellow_circle: 698：[划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)。自顶向下是搜索，自底向上是DP。[题解](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/solution/hua-fen-wei-kge-xiang-deng-de-zi-ji-by-leetcode/)又是状态压缩的动态规划，和473题火柴拼正方形类似。**值得二刷**。
1. :yellow_circle: 516：[最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)。自己写了一个求字符串s和s逆的最长公共子序列的方法，转化为了已知的问题。[题解](https://leetcode-cn.com/problems/longest-palindromic-subsequence/solution/zui-chang-hui-wen-zi-xu-lie-by-leetcode-hcjqp/)还提到了另外的动态规划方法。$dp[i][j]$代表从$i$到$j$的最长回文子序列，然后比较$s[i-1]$和$s[j+1]$。
1. :yellow_circle: 375：[猜数字大小II](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/)。**不是贪心**。一种经典的求解最优化的动态规划问题，还有一道扔鸡蛋的题目。[题解](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/solution/cai-shu-zi-da-xiao-ii-by-leetcode-soluti-a7vg/)中讲明了此题需要进行区间上的动态规划并求出极值。主要的状态转移方程为：$dp[i][j]=min(k+max(dp[i][k],dp[k][j]))，k \in [i,j]$。
1. :yellow_circle: 152：[乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)。很值得思考的题目。很明显和最大子数组和类似，但是问题在于存在0和负数。此时只需要同时保留max、min，就仍可以获得最终解。因为min中的负数可以再经一次和负数相乘变正。
1. :red_circle: 312：[戳气球](https://leetcode-cn.com/problems/burst-balloons/)。经典区间DP，模板$dp[l][r]=max/min/···(dp[l][r],f(dp[l][k],dp[k][r],g(l,k,r)))$。
1. :red_circle: 943：[最短超级串](https://leetcode-cn.com/problems/find-the-shortest-superstring/)。**经典字符串生成类**题目。从$O(N!)$降到$O(2^N)$的关键思路，在于并不需要计算全部的排列。[题解](https://leetcode-cn.com/problems/find-the-shortest-superstring/solution/zui-duan-chao-ji-chuan-by-leetcode/)只需要计算每一次以第j个字符串为结尾时，所能达到的最优长度。再结合对全部二进制子集的枚举。这种思路是通用的：用二进制枚举子集，然后计算当前这一步的最优解。
1. :red_circle: 44：[通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)。和正则类似的动态规划。但其实还有[贪心的题解](https://leetcode-cn.com/problems/wildcard-matching/solution/tong-pei-fu-pi-pei-by-leetcode-solution/)。
1. :red_circle: 72：[编辑距离](https://leetcode-cn.com/problems/edit-distance/)。字符串之间变换的DP方法都比较相似。$dp[i][j]$取最小值：$dp[i-1][j-1]$、$dp[i-1][j-1]+1$、$dp[i][j-1]+1$、$dp[i-1][j]$。对两个串前缀的各情况进行穷举：前缀最后一对儿字符相等，替换字符，添加一个字符，删除一个字符。
1. :red_circle: 887：[鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)。求给定鸡蛋数量和楼层总数情况下，测得安全楼层所需的最小次数。非常经典的题目，[题解](https://leetcode-cn.com/problems/super-egg-drop/solution/ji-dan-diao-luo-by-leetcode-solution-2/)提出了3种方法，从二分+动态规划，单调动态规划，再到数学法（**逆向思维**，将楼层作为目标值）。其中带有决策单调性优化的动态规划非常值得思考。**非常值得二刷**。
1. :red_circle: 546：[移除盒子](https://leetcode-cn.com/problems/remove-boxes/)。搜索式的DP写法。本题指出了一类相对复杂的区间DP问题，即区间结构可以由于之前的决策发生改变。使用普通的$[l,r]$不能表示出全部的区间结构。[题解](https://leetcode-cn.com/problems/remove-boxes/solution/yi-chu-he-zi-by-leetcode-solution/)指出这种情况下需要引入额外的状态维度，来表示出一次决策所使用的全部区间。如$[l,r,k]$代表范围后还有$k$个。**值得二刷**。
1. :red_circle: 664：[奇怪的打印机](https://leetcode-cn.com/problems/strange-printer/)。其实和546题是一模一样的。锻炼下举一反三的能力。
1. :red_circle: 514：[自由之路](https://leetcode-cn.com/problems/freedom-trail/)。自己写了个略带优化的搜索。实际可以用DP。比较特别的一道题目。$dp[i][j]$代表对齐$key[i]$时，位于$ring[j]$位置的最小移动步数。注意环是可以逆时针、顺时针转。
1. :red_circle: 741：[摘樱桃](https://leetcode-cn.com/problems/cherry-pickup/)。经典“多线程”DP，其实就是状态方程中同时考虑多个决策。$dp[point_1][point_2]$$=f(dp[prev(point_1)][point_2]$$,dp[point_1][prev(point_2)])$。
1. :red_circle: 85：[最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)。84题的变种，动态规划和单调栈的结合。[题解](https://leetcode-cn.com/problems/maximal-rectangle/solution/zui-da-ju-xing-by-leetcode-solution-bjlu/)是预处理+单调栈。
1. :red_circle: 689：[三个无重叠子数组的最大和](https://leetcode-cn.com/problems/maximum-sum-of-3-non-overlapping-subarrays/)。经典的**多种中间状态**输出到最终决策的动态规划。3次预处理，分别是每个位置的子数组和$sum[]$，i之前的子数组和最大值$before[i]$，j之后的子数组和的最大值$after[j]$。求最大值$sum[i]+before[i-k]+after[i+k]$。[题解](https://leetcode-cn.com/problems/maximum-sum-of-3-non-overlapping-subarrays/solution/san-ge-wu-zhong-die-zi-shu-zu-de-zui-da-4a8lb/)还有更厉害的$O(1)$空间的解法。
1. :red_circle: 1531：[压缩字符串II](https://leetcode-cn.com/problems/string-compression-ii/)。不容易的一道题目。有[反方向选取](https://leetcode-cn.com/problems/string-compression-ii/solution/dong-tai-gui-hua-shi-jian-on3kong-jian-on2-by-newh/)和[删除](https://leetcode-cn.com/problems/string-compression-ii/solution/ya-suo-zi-fu-chuan-ii-by-leetcode-solution/)两种题解。**值得二刷**。
1. :red_circle: 956：[最高的广告牌](https://leetcode-cn.com/problems/tallest-billboard/)。比较特别的一题。[题解](https://leetcode-cn.com/problems/tallest-billboard/solution/zui-gao-de-yan-gao-pai-by-leetcode/)两种解法，递归式DP方法$dp[i][s]$代表前i个管子正负总和为s时，正数和的最值。细节：实现时需要保证s>0，将总和**向正平移5000**，不影响结果。第二种是对半搜索，也比较巧妙，也值得一看。递归式DP其实和记忆化搜索非常相似。
1. :red_circle: LCP 57：[打地鼠](https://leetcode-cn.com/problems/ZbAuEH/)。团体赛题目。自己写了一个$O(nlogk),k=1e9$的dp，但其实可以做到$O(n)$。问题的关键在于理解，以时间向前回溯dp，和用步数向前回溯其实都是对的。但前者慢一些（常数太大了），步数其实最多只需要回溯45步（3 * 3棋盘，最多4步可到任意位置，3 * 3 * 4=45）。

###  哈希表
1. :green_circle: 219：[存在重复元素II](https://leetcode-cn.com/problems/contains-duplicate-ii/)。滑动窗口+哈希表。
1. :yellow_circle: 945：[使数组唯一的最小增量值](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/)。把哈希表的线性探测思路应用到算法题目中。[题解](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/solution/ji-shu-onxian-xing-tan-ce-fa-onpai-xu-onlogn-yi-ya/)还结合了路径压缩。
1. :yellow_circle: 694：[不同岛屿的数量](https://leetcode-cn.com/problems/number-of-distinct-islands/)。[非会员链接](https://www.jianshu.com/p/492db10d4159)。本题提供了一个散列化思路，网格散列（岛屿轮廓形状）、路径散列（DFS路径）。如果形状一样，则DFS路径必定相等。可以直接用相对坐标写为字符串，作为散列值。
1. :red_circle: 711：[不同岛屿的数量II](https://leetcode-cn.com/problems/number-of-distinct-islands-ii/)。[非会员传送门](https://blog.csdn.net/pianzang5201/article/details/93503836)。和694不同的地方在于这里会有旋转和翻转。其实这个就能很好的考验对于哈希的理解。只要换一个哈希思路，就能保证翻转、旋转不变性。即对形状坐标序列进行排序，并选择字典序最小的作为该形状的记录。

###  多指针 & 滑动窗口
1. :yellow_circle: 2516：[每种字符至少取K个](https://leetcode.cn/problems/take-k-of-each-character-from-left-and-right/description/)。自己写了对保留部分长度进行二分的。题解是滑动窗口（双指针）效率更高。
1. :green_circle: 26：[删除数组重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)。输入已经有序，单向双指针扫描。
1. :green_circle: 141：[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)。快慢指针。如果链表中存在循环，一定会相遇。
1. :green_circle: 27：[移除元素](https://leetcode-cn.com/problems/remove-element/)。从左右两侧分别开始，将需要移除的元素覆盖。
1. :yellow_circle: 3：[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)。
1. :yellow_circle: 15：[三数之和](https://leetcode-cn.com/problems/3sum)。排序加对撞指针（双指针相向运动，一般只有输入有序时这样用）。**值得二刷**
1. :yellow_circle: 16：[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest)。依然是排序加对撞。
1. :yellow_circle: 238：[除自身外的乘积](https://leetcode-cn.com/problems/product-of-array-except-self)。有的时候可以从复杂度要求思考方法。
1. :yellow_circle: 18：[四数之和](https://leetcode-cn.com/problems/4sum)。**值得二刷**。
1. :yellow_circle: 1151：[最少交换次数来组合所有的1](https://leetcode-cn.com/problems/minimum-swaps-to-group-all-1s-together/)。[非会员链接](https://blog.csdn.net/qq_17550379/article/details/99169544)。滑动窗口+前缀和。滑动窗口的关键之一就是如何设置窗口大小。
1. :yellow_circle: 424：[替换后的最长重复字符](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)。DP明显会超时。而本题带有明显的区间的特征，可以考虑使用双指针解法，详见[题解](https://leetcode-cn.com/problems/longest-repeating-character-replacement/solution/ti-huan-hou-de-zui-chang-zhong-fu-zi-fu-n6aza/)。注意其实当我们找到任意一种解之后，区间长度一定不会减小。
1. :red_circle: 30：[串联所有单词的子串](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)。只要想到是滑动窗口（因为匹配目标**长度固定**），其实就很好办了。使用双hash，一个统计窗口内单词数，一个统计words中的单词数。遍历多种滑动窗口的起点即可。
1. :red_circle: 32：[最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)。本题[题解](https://leetcode-cn.com/problems/longest-valid-parentheses/solution/zui-chang-you-xiao-gua-hao-by-leetcode-solution/)有动态规划、栈（括号匹配用栈非常好想）、双指针三种方案。其中贪心策略的双指针最为优越。括号的通用贪心策略：**右括号多于左括号时，一定非法**。

###  分治
1. :green_circle: 70：[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)。经典斐波那契问题。矩阵快速幂。
2. :yellow_circle: 215：[数组中第K个最大的元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)。可以使用[BFPRT算法]({{<relref "/content/post/CSBasic/algo4-sequenceCount.md#选择问题">}})，理论上更快，但是其实常数很高，也可以用小顶堆来实现，理论上复杂度高一些。
3. :yellow_circle: 148：[排序链表](https://leetcode-cn.com/problems/sort-list/)。进阶要求是$O(nlogn)$时间和$O(1)$的空间。只能使用自底向上的原地归并排序。

###  排序
1. :yellow_circle: 220：[存在重复元素III](https://leetcode-cn.com/problems/contains-duplicate-iii/)。使用滑动窗口+平衡树的方式可以很容易得到$O(nlogk)$的时间复杂度。但[题解](https://leetcode-cn.com/problems/contains-duplicate-iii/solution/cun-zai-zhong-fu-yuan-su-iii-by-leetcode-bbkt/)使用桶排序的思想来解决**abs值是否在一个区间内**的判断问题，需要判断当前桶和相邻桶。
1. :yellow_circle: 355：[设计推特](https://leetcode-cn.com/problems/design-twitter/)。思考的时候可以参考下基础算法题。本体实际上是合并k个有序链表。

###  位运算
1. :green_circle: 231：[2的幂](https://leetcode-cn.com/problems/power-of-two/solution/)。直接用n&n-1。
1. :yellow_circle: 201：[数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)。纯暴力遍历不可取。题解即求两个数字的[二进制下的公共前缀](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)，里面提到了用于去除二进制串最右侧1的，Brian Kernighan算法。
1. :yellow_circle: 320：[列举单词的全部缩写](https://leetcode-cn.com/problems/generalized-abbreviation/)。[非会员链接](https://blog.csdn.net/qq_21201267/article/details/107859696)。可以用**二进制运算**，每个bit位代表是否用数字进行缩写，并最终输出。实现上比回溯的效率高很多。当然本质上还是$O(2^n)$。
1. :red_circle: 1178：[猜字谜](https://leetcode-cn.com/problems/number-of-valid-words-for-each-puzzle/)。位运算+优化可以解决。32位以内的状态都可以很容易的用int的二进制位做存储，并和相同状态合并于一个内map计数。另外还利用$j=(j-1)&k$计算所有状态值k的二进制子集，由此利用谜底长度较少，子集数量较少的特性，快速遍历计数。详见[题解](https://leetcode-cn.com/problems/number-of-valid-words-for-each-puzzle/solution/cai-zi-mi-by-leetcode-solution-345u/)。

###  并发
1. :green_circle: 1114：[按序打印](https://leetcode-cn.com/problems/print-in-order/solution/)。条件变量、锁。

###  图论
1. :yellow_circle: 721：[账户合并](https://leetcode-cn.com/problems/accounts-merge/)。基础并查集题目。
1. :yellow_circle: 1162：[地图分析](https://leetcode-cn.com/problems/as-far-from-land-as-possible/)。很有代表性的一道题目，巧妙地把问题转换为多源点BFS。[题解](https://leetcode-cn.com/problems/as-far-from-land-as-possible/solution/di-tu-fen-xi-by-leetcode-solution/)中甚至还有更精妙的DP解法。**值得二刷**。
1. :yellow_circle: 207：[课程表](https://leetcode-cn.com/problems/course-schedule/)。拓扑排序。[wiki](http://en.wikipedia.org/wiki/Topological_sorting)。
1. :red_circle: 332：[重新安排行程](https://leetcode-cn.com/problems/reconstruct-itinerary/)。学习图论概念：[欧拉图&半欧拉图&欧拉回路&半欧通路](https://oi-wiki.org/graph/euler/)。[Hierholzer算法题解](https://leetcode-cn.com/problems/reconstruct-itinerary/solution/zhong-xin-an-pai-xing-cheng-by-leetcode-solution/)。**值得二刷**。
1. :yellow_circle: 310：[最小高度树](https://leetcode-cn.com/problems/minimum-height-trees/)。自己写的是普通的BFS。[题解]()中揭示了本题实际上是逐步删除度为1的节点，即拓扑排序。或者说这是一类，两端烧香求中点的题目。
1. :yellow_circle: 743：[网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)。基础款单源有向最短路。邻接表+djikstra+堆优化。
1. :yellow_circle: 787：[K站中转内最便宜的航班](https://leetcode-cn.com/problems/cheapest-flights-within-k-stops)。K轮[Bellman-Ford算法]({{<relref "/content/post/CSBasic/algo6-graph.md#单源最短路">}})。
1. :red_circle: 685：[冗余连接II](https://leetcode-cn.com/problems/redundant-connection-ii/solution/)。树+并查集。根据冗余连接的不同情况，查找可以断开的边。
1. :red_circle: 1168：[水资源分配优化](https://leetcode-cn.com/problems/optimize-water-distribution-in-a-village/)。[非会员传送门](https://blog.csdn.net/qq_17550379/article/details/100070671)。图论的经典思路：**添加超级源点**。通过引入超级源点，水井费用也成为一种普通的边。转化后就成为了最小生成树题目。
1. :red_circle: 329：[矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)。经典转化思路：将原题目中的一些约束，修改为图论中点和点之间的边关系。而最长路径恰好可以用拓扑排序的方式进行计算（或者说是类似BFS的方式）。
1. :red_circle: 675：[为高尔夫比赛砍树](https://leetcode-cn.com/problems/cut-off-trees-for-golf-event/)。思路很简单，排序后找最短路。但复杂度也奇高。[官方题解](https://leetcode-cn.com/problems/cut-off-trees-for-golf-event/solution/wei-gao-er-fu-bi-sai-kan-shu-by-leetcode/)和[民间题解](https://leetcode-cn.com/problems/cut-off-trees-for-golf-event/solution/c-160ms-ti-jie-xi-shuo-chang-shu-you-hua-na-xie-sh/)都提到了优化问题，尤其是**间隔搜索**值得思考。更厉害的是[分治法优化思路](https://leetcode-cn.com/problems/cut-off-trees-for-golf-event/solution/on3fen-zhi-c-48ms-100-by-hqztrue-jhxz/)。**值得二刷**。
1. :red_circle: 854：[相似度为K的字符串](https://leetcode-cn.com/problems/k-similar-strings/)。又是字符串变换转为图论变换的题目。本题的[题解](https://leetcode-cn.com/problems/k-similar-strings/solution/xiang-si-du-wei-k-de-zi-fu-chuan-by-leetcode/)把字符移动理解为对边的变换。
1. :red_circle: 1197：[进击的骑士](https://leetcode-cn.com/problems/minimum-knight-moves/)。[非会员传送门](https://blog.csdn.net/qq_17550379/article/details/101195668)。使用[A*算法](https://zhuanlan.zhihu.com/p/54510444)，给搜索添加启发式估计值作为排序规则的一部分。这个规则的主要目的是，不让100%恶化的情况入队。
1. :red_circle: 2493：[将节点分成尽可能多的组](https://leetcode.cn/problems/divide-nodes-into-the-maximum-number-of-groups/description/)。本质上是求若干个连通子图各自的图直径。图的直径的算法就是两次BFS，第一次随机，第二次从第一次BFS最后入队节点开始。可以先用并查集对图做分割。
2. :red_circle: 2603: [收集树中金币](https://leetcode.cn/problems/collect-coins-in-a-tree/description/)。看了题解，核心是对图做变换：剪枝、去叶子节点。**思考最终路径的规则**：任何一条路径最多只需要走两次，路径终点到（最远的）有金币的节点距离为2。因此变换有两条：递归删除所有无金币的叶子节点，对最终树进行2次删除叶子节点的操作。最终剩下的边，每个都会遍历。

###  数据结构
1. :red_circle: 2179：[统计数组中好三元组数目](https://leetcode-cn.com/problems/count-good-triplets-in-an-array/)。核心题解思路是，以变量y遍历第一个列表，视作三元组的中间元素，并统计第二个列表中，位于y前面的变量中，有多少也在第一个列表中y位置之前出现过。这个统计的信息，恰好可以用[树状数组](https://zhuanlan.zhihu.com/p/93795692)来进行维护。由此达到$O(nlogn)$。
1. :green_circle: 496：[下一个更大元素I](https://leetcode-cn.com/problems/next-greater-element-i/)。自己写的是$O(n^2)$，题解中推荐使用单调栈。
1. :yellow_circle: 731：[我的日程安排]。看了题解，第一种是可以用两个set，分别存储无重叠时间段，和有一重重叠的时间段，新加入时间段不允许和一重重叠的时间段再重叠。另有题解[边界计数](https://leetcode-cn.com/problems/my-calendar-ii/solution/wo-de-ri-cheng-an-pai-biao-ii-by-leetcode/)，[线段树基础题型](https://leetcode-cn.com/problems/my-calendar-ii/solution/xian-duan-shu-dong-tai-kai-dian-lan-duo-336be/)，**值得二刷**。
1. :yellow_circle: 654：[最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)。单调栈。
1. :yellow_circle: 1395：[统计作战单位数](https://leetcode-cn.com/problems/count-number-of-teams/)。暴力枚举中间值能过。学习以下[题解](https://leetcode-cn.com/problems/count-number-of-teams/solution/tong-ji-zuo-zhan-dan-wei-shu-by-leetcode-solution/)使用离散化+树状数组高效解决。这里用的离散化方法是通过排序，确定其序号作为关键字。
1. :yellow_circle: 109：[从有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)。锻炼基础能力，熟悉前中后序遍历。**值得二刷**。
1. :yellow_circle: 146：[LRU缓存](https://leetcode-cn.com/problems/lru-cache/)。LRU缓存是最近最少使用，按照使用时间戳排序。用一个链表，加一个哈希表即可。
1. :red_circle: 84：[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)。经典**单调栈**题目。
1. :red_circle: 42：[接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)。单调栈。思考的时候可以从三种基本简单情况开始：[2,1,2]，[2,1,3]，[3,1,2]。单调栈其实就是保留有用的单调信息，在出入栈的时候再进行统一计算的一种方法。
1. :red_circle: 1157：[子数组中占绝大多数的元素](https://leetcode-cn.com/problems/online-majority-element-in-subarray/)。**线段树**。树中维护的数据是BM众数投票算法的值。重点在于学会[线段树](https://www.cnblogs.com/AC-King/p/7789013.html)。另外还有题解提到[树套树]解法(https://leetcode-cn.com/problems/online-majority-element-in-subarray/solution/wu-nao-shu-ju-jie-gou-zuo-fa-shu-tao-shu-by-wnjxyk/)。
1. :red_circle: 23：[合并k个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)。本题实际上是使用一个k个大小的**小顶堆**，每次出堆元素所在的链表继续入堆。
1. :red_circle: 1354：[多次求和构造目标数组](https://leetcode-cn.com/problems/construct-target-array-with-multiple-sums/)。**逆向思维**，求生成的可能性很多，但是反方向的恢复方式只有一个。使用大顶堆，每次消去当前数组最大的数字，测试是否能反向恢复数组至初始状态。
1. :red_circle: 432：[全O(1)的数据结构](https://leetcode-cn.com/problems/all-oone-data-structure/)。哈希表+链表。思路很简单。但是在prev、next、swap、iter_swap这方面受了半天苦。
1. :red_circle: 460：[LFU缓存](https://leetcode-cn.com/problems/lfu-cache/)。结合多种数据结构：散列表、链表、最小频率标记。按照频率设置不同链表存储键值是本题的精髓。
1. :red_circle: 99：[恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)。还是中序Morris遍历。
1. :red_circle: 440：[字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/)。数字顺序类题目大都需要思考顺序和数量之间的规律。自己的思路比较繁琐，是按位去生成的，一个数字除最高位是1~9，其他位均为0\~9。数量可以逐位累加（如1是最高位的数字每一位分别可以有1，10，100，...）。[题解](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/solution/zi-dian-xu-de-di-kxiao-shu-zi-by-leetcod-bfy0/)更为简洁，建议学习。题解思路是从思考字符串的字典树构造开始，按照对树的遍历进行。
1. :red_circle: 862：[和至少为K的最短子数组](https://leetcode-cn.com/problems/shortest-subarray-with-sum-at-least-k/solution/he-zhi-shao-wei-k-de-zui-duan-zi-shu-zu-by-leetcod/)。结合多种思路，前缀和（加速）+单调栈（因为有负数，不能简单滑动窗口）+二分（加速）。**值得思考**。
1. :red_circle: 1606：[找到处理最多请求的服务器](https://leetcode-cn.com/problems/find-servers-that-handled-most-number-of-requests/)。典型的多种数据结构配合。思考思路：需要有一个数据结构有序保存服务器当前任务结束时刻，还需要有一个数据结构有序保存空闲服务器序号。思维上**不要局限于单一数据结构**解决所有需求。
1. :red_circle: 732：[我的日程安排表III](https://leetcode-cn.com/problems/my-calendar-iii/)。经典线段树板子，需要动态增加节点。即抛弃build过程，在update过程中（具体就是push_down）过程中，进行原来的节点分裂。
2. :red_circle: 2736：[最大和查询](https://leetcode.cn/problems/maximum-sum-queries/description/)。看了[题解](https://leetcode.cn/problems/maximum-sum-queries/solutions/2530255/jian-dan-bao-li-xian-duan-shu-by-ketus82-tes0/)。还是线段树。**同一下标的多维度的值也可以用线段树进行区间维护**。线段树**并不要求区间之间一定满足什么关系**，看题目内容，可能要遍历很多的区间，必要的时候可以通过剪枝。其实仔细思考能发现本题的几个要点：和的极大值可以通过区间维护，两个加数的极小值/极大值也可以通过区间维护。如此就能通过排除

###  其他
1. :green_circle: 665：[非递减数列](https://leetcode-cn.com/problems/non-decreasing-array/)。不简单的简单题，**值得二刷**。考虑情况要完整。
1. :yellow_circle: 78：[求全部子集](https://leetcode-cn.com/problems/subsets)。使用vector一定减少内存重分配。
1. :yellow_circle: 169：[找众数](https://leetcode-cn.com/problems/majority-element)。Boyer-Moore投票算法。维护当前众数和计数器。相等+1，否则-1。变为0则更换数字。
1. :yellow_circle: 287：[寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)。要求$O(n)$复杂度。题解使用等价变换+[Floyd判圈法](https://www.jianshu.com/p/36a89d938440)，**值得二刷**。
1. :yellow_circle: 138：[复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)。题解方法非常有趣，原地将链表复制一遍，变为a->a'->b->b'->...->nil。然后再对random域赋值。这种原地操作的思路值得思考。
1. :red_circle: 41：[缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)。题目给出了$O(n)$时间，$O(1)$的空间限制。但是**输入数组**是可以使用的空间。由于寻找正整数，所以忽略负数。**原地排序新思路**，一个萝卜一个坑，把数字移动到自己的位置上，最后遍历看哪里的下标不是自己。
1. :yellow_circle: 29：[两数相除](https://leetcode-cn.com/problems/divide-two-integers/)。非常基础的题目，一点快速乘的思想就可以。但是非常考验对于**边界溢出**情况的考虑。由于负数范围大于正数，实际上我更推荐把数字**全转为负数**再进行运算和分类讨论（而不是转为正数），转为无符号也行。另外最好单独处理INT_MIN/1和INT_MIN/-1这两种情况。
1. :yellow_circle: 50：[Pow(x,n)](https://leetcode-cn.com/problems/powx-n/)。快速幂，但n是int全范围。**注意INT_MIN**。
1. :red_circle: 780：[到达终点](https://leetcode-cn.com/problems/reaching-points/)。典型的逆向思维的题。当题目中有以初始状态，变换到结束状态这种描述时，就可以思考是否从结束到初始进行计算是更为简便的。