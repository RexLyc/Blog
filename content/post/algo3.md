---
title: "算法导论其三：排序（施工中）"
date: 2021-07-17T15:11:40+08:00
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
排序问题是编程中的重要基石，排序中用到的很多思想也非常有借鉴价值。
<!--more-->
# 对比
|  名称   | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度  | 是否稳定 |
|  ----  | ----  |  ----  | ----  |  ----  |
|  插入排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
|  希尔(Shell)排序 | $O(n^{1.3})$ | $O(n^2)$ | $O(1)$ | 不稳定 |
|  堆排序 | $O(nlogn)$ | $O(nlogn)$ | $O(1)$ | 不稳定 |
|  冒泡排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
|  快速排序 | $O(nlogn)$ | $O(n^2)$ | $O(logn)$ | 稳定 |
|  归并排序 | $O(nlogn)$ | $O(nlogn)$ | $O(n)$ | 稳定 |
|  基数排序 | $O(d(r+n))$ | $O(d(r+n))$ | $O(rd+n)$ | 稳定 |
|  桶排序 | $O(nlog\frac{n}{k})$ | $O(nlog\frac{n}{k})$ | $O(n)$ | 稳定 |
|  计数排序 | $O(n)$ | $O(n)$ | $O(n)$ | 非稳定 |

注：其中桶排序数据存疑。

# 基于比较的
- ## 插入排序
- ## 归并排序
- ## 堆排序
- ## 快速排序
    1. 非常经典的算法，分治法的杰出代表。虽然有小瑕疵，但经过改造目前仍然被广泛应用。
    2. 步骤：
        1. Partition!根据主元将序列分为两部分，一部分小于主元，一部分大于主元。
        2. 同理分别处理两部分
    ```cpp
    //很妙的单向扫描写法（循环不变式：i的下一个是大于pivot）
    template<typename ForwardIt>
    void quick_sort(ForwardIt begin, ForwardIt end) {
        //partition
        auto i = begin, j = begin + 1;
        auto pivot = *begin;
        for (;j!=end;++j) {
            if (*j < pivot) {
                std::iter_swap(++i, j);
            }
        }
        std::iter_swap(i, begin);
        //递归
        if (begin < i)
            quick_sort(begin, i);
        if (i + 1 < end)
            quick_sort(i + 1, end);
    }
    ```
    3. 其中，选择主元并划分的步骤，就叫做Partition，这种思想经常用到。
- ## 随机化快速排序
    1. 快排的经典问题，在于非随机化的主元选取方式，会导致一些“恰好”的攻击例子，降低时间效率到$O(n^2)$。
        - 比如每次选取第一个为主元，则输入已经按升序排列好，那么等价于选择排序。
    2. 使用随机函数，随机化选取主元，移动主元到开始位置，复用上面的quick_sort。
# 基于计数
- ## 基数排序
- ## 桶排序
- ## 计数排序

# 额外
- ## 希尔排序