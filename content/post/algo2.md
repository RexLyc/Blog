---
title: "算法导论其二：分治（施工中）"
date: 2021-07-16T14:49:40+08:00
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
分治法是诸多算法的基本思想，把大问题分解为小问题，依次解决并最终合并。
<!--more-->
# 经典案例
1. 二分查找
    - 对有序序列查找指定元素：
        1. 比较序列中间元素和目标值
        2. 递归查找左侧、右侧
        3. 找到时直接返回
    - 代码(C++)
        ```cpp
        // std::upper_bound（首个大于value）可能的实现
        template<class ForwardIt, class T>
        ForwardIt upper_bound(ForwardIt first, ForwardIt last, const T& value)
        {
            ForwardIt it;
            typename std::iterator_traits<ForwardIt>::difference_type count, step;
            count = std::distance(first,last);
            while (count > 0) {
                it = first; 
                step = count / 2; 
                std::advance(it, step);
                //替换为if(*it<value) 则是lower_bound（首个不小于value）
                if (!(value < *it)) {
                    first = ++it;
                    count -= step + 1;
                }
                else {
                    // 这里需要理解，虽然看起来当前*it被排除在范围外了，但仍可能是最终结果
                    count = step;
                }
            }
            return first;
        }
        ```
    - 时间复杂度：$T(n)=T(n/2)+\Theta(1)$ 也即 $T(n)=\Theta(logn)$。
2. 快速幂运算
    - 求x的n次方
        1. 求$x^{\frac{n}{2}}$次方
        2. 求$x^{\frac{n}{2}} * x^{\frac{n}{2}}$
    - 代码(C++)
        ```cpp
        int pow(int x, int n){
            int result = 1;
            // 自底向上非递归写法
            // 拆分因子好理解一些
            while (n != 0) {
                if (n & 1) {
                    result *= x;
                }
                x *= x;
                n >>= 1;
            }
            return result;
        }
        ```


# 参考资料
1. 部分代码来自于[CppReference](https://zh.cppreference.com/w/%E9%A6%96%E9%A1%B5)