---
title: "编程题笔记：总结篇"
date: 2022-03-29T10:28:30+08:00
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
本篇以记录实用算法和技巧为主，和题目记录对照使用。
<!--more-->
## 常用版子
1. 二分
    - 参考STL实现：[lower_bound](https://zh.cppreference.com/w/cpp/algorithm/lower_bound)、[upper_bound](https://zh.cppreference.com/w/cpp/algorithm/upper_bound)
1. Partition
    - 参考STL实现：[partition](https://en.cppreference.com/w/cpp/algorithm/partition)

## 巧妙小算法
1. 弗洛伊德判圈法
    - 说明：一个可以在有限状态机、迭代函数、链表等算法或数据结构中判断是否存在环，以及判断环起始位置、长度的算法
    - 原理：用两个以不同速度前进的指针遍历，如果两个指针在结束前相遇，则必定存在环。此时分别在初始位置、相遇位置额外构造2个指针，以相同速度遍历，这一次相遇点一定是环的入口处。
    - 参考证明：[龟兔赛跑算法](https://blog.csdn.net/mq2553299/article/details/104045740)

1. 分割平面与空间公式
    - 直线划分平面：已有n-1条直线，新增第n条直线和n-1条线最多形成n-1个交点，这些焦点将新增直线划分为2个射线和n-2个线段，该直线外侧即本次新增直线后，新增加的平面区域数量，恰好和射线、线段一一对应，因此有：
        $f(n)=f(n-1)+2+(n-2)=f(n-1)+n,f(1)=2$
    - 折线划分平面：一个折线是形如五角星的一个角（如$<$）。已有n-1条折线，新增第n条折线，和n-1条折线（攻击2*(n-1)条射线），最多形成4*(n-1)个线段（折线顶点处不再有射线），和2个射线，但折线顶点处多算了一个，因此有：
        $f(n)=f(n-1)+4(n-1)+2-1=f(n-1)+4n-1,f(1)=2$
    - 平面分割空间：分割最大化时，相当于每一个平面需要和之前的所有平面相交，此时我们可以想象到，该平面上将会被若干交线划分为若干平面区域，该区域数量就恰好是新增的空间数量，联系前述公示，可有：
        $h(n)=h(n-1)+f(n-1)$
1. Morris遍历和线索二叉树
    - 参考：https://blog.csdn.net/softwarex4/article/details/95986102 ，https://www.jianshu.com/p/d2059062efac
1. 大数乘法：
    - 小于10w数据可以使用竖式乘法，大于10w可以使用快速傅里叶变换(FFT)
    - FFT关键词：
        - 参考：[MIT公开课（分治法和FFT）](https://www.youtube.com/watch?v=iTMn0Kt18tg) 、[博客园-快速傅里叶变换（FFT）详解](https://www.cnblogs.com/wangyh1008/p/9325715.html)、http://picks.logdown.com/posts/177631-fast-fourier-transform、以及[一个比较好的迭代](https://leetcode-cn.com/problems/multiply-strings/solution/fftjie-fa-by-kuai-xue-shi-qing-2/)
    - 参考：[大数乘法高效算法](https://blog.csdn.net/u010983881/article/details/77503519)
1. 字符串匹配：
    - 总结：
        - KMP：构造next数组，最长的相等前缀、后缀
        - BM：过于复杂，而且很长
        - Sunday：启发式，好写又好用，两个条件：适配时先看目标串的下一个待匹配字符，如果不存在于模式串中，则直接大跳，否则移动模式串到最右侧第一个能匹配到该待匹配字符的位置。
    - 参考：
        - [BF、KMP、BM、Sunday详解](https://www.cnblogs.com/Syhawk/p/4077295.html)
        - [从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)
        - [BF、RK、BM、KMP、Trie树、AC自动机](https://blog.csdn.net/weixin_40805537/article/details/89044710)
1. 回文字符串Manacher算法：
    - 概述：$O(n)$时间求字符串的全部回文子串
    - 核心原理：当前中心C，右边界R，待求$p[i]$，关于C镜像$p[2*C-i]$，只有当镜像回文长度会超出当前C的回文的左边界，或者直接赋值镜像p回文长度会超过右边界R，才需要中心扩展法
    - 参考：[马拉车算法详解](https://zhuanlan.zhihu.com/p/70532099)
1. 正则表达引擎
    - 参考：[正则表达引擎的原理](https://www.cnblogs.com/snake-hand/p/3153396.html)
1. 位运算技巧
    - 位运算不保证跨平台通用，实际生产中使用时需要注意
    - 无临时变量交换两个数的值：异或^
    - 二进制下统计1的个数、去掉最低位的1：n&(n-1)
    - lowbit，保留最低位的1：n&-n。
    - 查表法：统计1的个数（分四个字节）、计算奇偶校验位
    - 子集枚举
        - 其中：针对生成n元集合的k元子集，可用Gosper's Hack优化。能够在$O(1)$时间内获得升序的下一个子集二进制串。
    - 寻找一个/两个仅出现一次的数（其余两次）：异或^
    - 其实很多位运算在GCC中已经有内建函数：```__builtin_ctz```、```__builtin_clz```、```__builtin_parity```等
    - 更多技巧请参考：[位运算](https://blog.csdn.net/deaidai/article/details/78167367?utm_source=distribute.pc_relevant.none-task)、[位运算奇技淫巧](https://blog.csdn.net/holmofy/article/details/79360859)。
1. 组合与排列
    - C++中提供了```next_permutation```，按字典序进行全排列，获取下一个排列（直到最小字典序），基本思路如下
        1. 找到尾部的最长降序序列
        2. 将最长降序序列前的第一个值$A$，和序列中第一个比$A$大的值交换
        3. 翻转现在尾部的最长降序序列
        > 整体思路很简单，就是从尾部构造一个刚好比当前序列字典序更大的下一个序列。
1. 布隆过滤器：
    - 对于海量数据，判断一个数据：是否一定不在集合中，或者是否可能在集合中。即要么能100%确定数据一定不存在，要么就有一定误判率确定数据存在。
    - 基本原理：使用若干二进制位（一个bit向量）存储标记值，使用一组哈希函数，每个哈希函数都能映射任意一位，将其置为1，那么此时
        - 如果待检测数据，使用所有哈希函数后都存在某一个哈希输出的指定bit找不到“1”标记，说明该数据一定未录入
        - 如果待检测数据，使用哈希函数后都能找到“1”标记，则该数据可能已经录入
1. 差分数组
    - 理解差分数组的核心在于，理解差分数组是前缀和的逆运算。
    - 进阶：多次差分。但一个区间的值的修改，满足一个一次函数规则，那么可以用二次差分来累计区间修改。即对差分数组求差分数组。
    - 应用场景：
        - 求和、求前缀和很困难、很耗时。那就尝试先求出差分数组
    - 例子：
        - 对一个区间上的所有值+1，相比于暴力的对每一个值+1，或者用线段树，此时只需要用差分数组在区间首尾分别+1/-1，简单有效。
        - 对一个二维矩形内的所有值进行修改，此时更明显，暴力的每个值+1，或者用线段树都非常麻烦，此时用差分数组在矩形的四个角分别+1/-1，简单有效
1. LIS最长递增子序列
    - $O(n^2)$的并不难写，但是这类非常基础的问题是有$O(nlogn)$写法的。
    - 思路是：
        - 顺序遍历每一个数字
        - 尝试将其按大小顺序放入数组$d$中，利用二分找到位置
            - 如果放置位置在数组$d$的尾部，则记录当前长度
            - 如果放置位置在中间，直接替换不做处理
    - 证明：实际上是直接构造出了最长递增子序列。只不过数组$d$中任意时刻中，某一个位置上值不一定真的在该最长子序列内，但是一定有一个历史版本是正确的。如果有需要可以使用可持久化数据结构。
    - 代码示例
        ```cpp
        int lengthOfLIS(vector<int>& nums) {
            if(nums.empty())
                return 0;
            vector<int> dp(nums.size(),0);
            size_t len=0;
            for(auto t:nums){
                auto pos = lower_bound(dp.begin(),dp.begin()+len,t);
                *pos = t;
                if(pos==dp.begin()+len){
                    len++;
                }
            }
            return len;
        }
        ```
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
2. 对于动态开点线段树等，在运行时会创建大量对象的程序，可以在使用中引入一点点池化的思路，具体可见[题目](https://leetcode.cn/problems/count-integers-in-intervals/solutions/1495396/by-endlesscheng-clk2/comments/2195672)，简而言之
    ```cpp
    class YourSolution {
    public:
        static bool IsPooled;
        static YourSolution *poolInstance;

        void* operator new(size_t size){
            if(!isPooled){
                poolInstance = new YourSolution[1e5]; // 或者你想要的大小
                isPooled = true;
            }
            // 很不负责任的分配方式，但是做题够用了
            return pollInstance++;
        }

        void* operator delete(void *prt) noexcept {};

        // ... 剩余的内容
    }

    bool YourSolution::IsPooled;
    YourSolution* YourSolution::poolInstance = nullptr;
    ```

## 经典书籍
1. 《程序员面试经典》
    - 08.07，全排列的生成：可以去看[C++的next_permutation的实现](https://zh.cppreference.com/w/cpp/algorithm/next_permutation)。标准库主要是对```reverse_iterator```的使用非常精妙。
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
1. [图算法总结](https://www.cnblogs.com/Ace-Monster/p/9439557.html)
2. 强烈推荐[OI Wiki](https://oi-wiki.org/)，推荐其中关于数据结构、图论、计算几何的部分。