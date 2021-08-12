---
title: "算法导论其四：中位数和顺序统计"
date: 2021-07-24T16:13:42+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
求中位数是非常经典，且有相当精妙算法的一个问题。代表了顺序统计问题的思考方式。
<!--more-->
# 问题描述
- 中位数：数组排序后，位于中点的数字（总数奇数即为一个数字，否则有2个）。
- 顺序统计问题（选择问题）：求包含n个数字的集合中的第i小的数字，$1 \le i \le n$。第i个数字也叫做第i顺序统计量。
# 思路
- 暴力想法：排序，排序之后任意顺位的数字都不是问题。复杂度$\mathrm{O}(nlogn)$。
    - 面试官：“那有没有更快的呢？”
- 优化：实际上求排序相当于获取了过多的细节。降低时间复杂度的办法就是**减少信息获取**。
# 同时求最大最小值
- 暴力想法：每个数字比较两次，即和存储的最大和最小值都比较。比较次数$2n-2$。
- 优化想法：成对比较，每次取两个数字，并取两个数字中的小者和当前最小值比较，大者和当前最大值比较。比较次数至多$3 \lfloor n/2 \rfloor$。
- 渐进时间$\Theta(n)$
- 为什么要这么吝啬：常数项其实影响很大（这里就差出去1.5倍），别让自己总是无意识的写出很暴力的算法。
# 选择问题
- 随机选择算法
    - 很好理解的一个**期望**时间复杂度为$\Theta(n)$的算法（注意期望的含义哈，有随机在里面了，还不得有点幺蛾子）
    - 思路概述：在排序章节，我们学习了随机化快速排序，其核心思想是随机选取主元，并用主元将当前片段$A[l,r]$分片（partition操作）。随机选择算法利用了这一过程，在求第i小元素时，先进行分片，可以很容易的得知当前主元的位置p，$l\le p \le r$。若$p==i$，则问题结束。否则根据p和i的大小分别选择分片的前半段和后半段进行递归。
    ```cpp
    // 传统单向扫描
    template<typename iter>
    iter partition(iter begin, iter end, iter pivot) {
        std::iter_swap(begin, pivot);
        iter i, j;
        for (i = begin, j = begin + 1; j != end; ++j) {
            if (*j < *begin) {
                ++i;
                std::iter_swap(i, j);
            }
        }
        std::iter_swap(i, begin);
        return i;
    }

    // 调试用的，留给你啦
    template<typename iter>
    void iterPrint(iter begin, iter end) {
        for (auto it = begin; it != end; ++it) {
            std::cout << *it << " ";
        }
        std::cout<<std::endl;
    }

    // randomized - select
    template<typename iter>
    iter randomizedSelect(iter curBegin, iter curEnd, size_t i) {
        // 使用了C++11中简单好用的mt19937随机数，用chrono库获取时间作为种子
        static std::size_t seed = std::chrono::system_clock::now().time_since_epoch().count();
        static std::mt19937_64 rand(seed);
        if (curBegin == curEnd) {
            return curBegin;
        }
        // 随机数范围是0到区间最后一位，构造函数就是给出这个闭区间。
        std::uniform_int_distribution<std::size_t> dist(0, std::distance(curBegin, curEnd)-1);
        auto pIndex = curBegin;
        std::advance(pIndex, dist(rand));
        //std::cout << std::distance(curBegin, pIndex) << std::endl;
        pIndex = partition(curBegin, curEnd, pIndex);
        //std::cout << std::distance(curBegin, pIndex) << std::endl;
        //iterPrint(curBegin, curEnd);
        if (std::distance(curBegin,pIndex)== i) {
            return pIndex;
        }
        else if (std::distance(curBegin, pIndex) < i) {
            return randomizedSelect(pIndex + 1, curEnd, i - std::distance(curBegin, pIndex)-1);
        }
        else {
            return randomizedSelect(curBegin, pIndex, i);
        }
    }
    ```
- BFPRT算法
    - 这个非常炫酷的，**最坏**时间复杂度$\Theta(n)$的算法，由Blum、Floyd、Pratt、Rivest、Tarjan发明。巧妙到虽然可能写不出来，但是看过就一定能有印象（指对算法很巧妙这件事情有印象）的程度。
    - 普通分片和随机化分片的最大问题，无法保证主元选取得很好。BFPRT主要解决的就是这个问题。
    - 步骤：
        1. 将数据分为$\lceil n/5 \rceil$组，除1个组外，其他组全部是5个数。
        2. 每一组求中位数
        3. 对每一组的中位数求中位数（从第1步开始递归）
        4. 用中位数的中位数对当前数组分片，得知该中位数所在位置为k
        5. 和随机选择算法同理，比较k和选择值i，分情况递归。
    - 算法在存在双中位数情况下，默认取小的那个。
    ```cpp
    // partition和iterPrint在上面，自取
    // 使用插入排序获得中位数
    template<typename iter>
    iter GetMedian(iter begin, iter end) {
        // insert sort
        iter a, b;
        for (std::size_t i = 1; i != std::distance(begin, end); ++i) {
            for (std::size_t j = i; j != 0; --j) {
                a = b = begin;
                std::advance(a, j - 1);
                std::advance(b, j);
                if (*a>*b) {
                    std::iter_swap(a,b);
                }
                else {
                    break;
                }
            }
        }
        //iterPrint(begin, end);
        std::advance(begin, (std::distance(begin, end) - 1) / 2); // 默认取较小的中位数
        //std::cout << "choose local median: " << *begin << std::endl;
        return begin;
    }

    template<typename iter>
    iter BFPRT(iter curBegin, iter curEnd, std::size_t i) {
        typedef std::iterator_traits<iter>::value_type T;
        if (std::distance(curBegin,curEnd)==1) {
            return curBegin;
        }
        std::vector<T> medians;
        medians.reserve((std::distance(curBegin, curEnd) + 4) / 5); // 保证不完整的一组也考虑进来
        iter it = curBegin;
        iter temp = it;
        while (std::distance(it, curEnd) >= 5) {
            std::advance(temp, 5);
            medians.push_back(*GetMedian(it, temp));
            it = temp;
        }
        if (it != curEnd) { // 这就是那不完整的一组
            medians.push_back(*GetMedian(it, curEnd));
        }
        iter pivot = BFPRT(medians.begin(), medians.end(), medians.size() / 2);
        pivot = std::find(curBegin, curEnd, *pivot); //找到原来容器中的位置
        pivot = partition(curBegin, curEnd, pivot); //找到分片之后的位置（这里暂时没想到更好的办法）
        //iterPrint(curBegin, curEnd);
        if (std::distance(curBegin, pivot) == i) {
            return pivot;
        }
        else if (std::distance(curBegin, pivot) < i) {
            return BFPRT(pivot + 1, curEnd, i - std::distance(curBegin, pivot) - 1);
        }
        else {
            return BFPRT(curBegin, pivot, i);
        }
    }
    ```
    - 为什么是$\Theta(n)$?
        - 关键图解（可以去看一眼视频课，这里讲的比较好）
        ![pic](/images/algoSeries/BFPRT.svg)
        - 从上图可以知道，求中位数的中位数x后，实际上将原有数据分为了4个部分，左上角的一定不比x大的区域，右下角的一定不比x小的区域。
        - 因此我们可以认为，小于等于x的元素至少有 $3(\lceil \frac{1}{2} \lceil \frac{n}{5} \rceil \rceil -2) \ge \frac{3n}{10} -6$
            - 注意这里为了谨慎（并且好算），我们去掉x所在的组，和假设有可能会多出来的不完整一组，里面$-2$了。
        - 最坏情况下，我们要去递归更多的一半，此时很容易知道$T(n)=T(\frac{n}{5})+T(\frac{7n}{10}+6) + \Theta(n)$
            - 第一项是递归处理各组的中位数，求出中位数的中位数；第二项是求顺序统计量的递归；第三项是当前统计各组中位数的花费。
            - 使用代换法就能够获得严格证明，即令$T(n)\le cn$，c需要是一个足够大的常数。***最妙的地方来啦*** : $\frac{n}{5}+\frac{7n}{10} \lt n $
# 尾声
&emsp;&emsp;实际上，BFPRT理论意义大于实际意义，这个最坏线性时间的算法，常数项太大了（代码也多）。随机化选择算法其实是你更好的选择啦。