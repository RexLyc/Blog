---
title: "算法导论其三：排序"
date: 2021-07-17T15:11:40+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
排序问题是编程中的重要基石，排序中用到的很多思想也非常有借鉴价值。
<!--more-->
# 对比
|  名称   | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度  | 是否稳定 |
|  ----  | ----  |  ----  | ----  |  ----  |
|  选择排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
|  插入排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 数组不稳定，链表稳定（根据实现） |
|  希尔(Shell)排序 | $O(n^{1.3})$ | $O(n^2)$ | $O(1)$ | 不稳定 |
|  堆排序 | $O(nlogn)$ | $O(nlogn)$ | $O(1)$ | 不稳定 |
|  冒泡排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
|  快速排序 | $O(nlogn)$ | $O(n^2)$ | $O(logn)$ | 不稳定 |
|  归并排序 | $O(nlogn)$ | $O(nlogn)$ | $O(n)$ | 稳定 |
|  基数排序 | $O(d(r+n))$ | $O(d(r+n))$ | $O(rd+n)$ | 稳定 |
|  桶排序 | $O(nlog\frac{n}{k})$ | $O(nlog\frac{n}{k})$ | $O(n)$ | 稳定 |
|  计数排序 | $O(n)$ | $O(n)$ | $O(n)$ | 稳定 |

注：其中桶排序数据存疑。

# 基于比较的
- ## 插入排序
    1. 步骤：
        1. 取未排序区间的第一个数
        2. 从后向前，一边移动已排序区间，一边判断能否在此位置插入
        3. 重复直到结束
    2. 变种-希尔(Shell)排序：选择合适的增量序列，甚至能够在最坏情况下降低复杂度
        1. 选择一个gap（如length/2)，将数组按照gap划分{0,gap,2\*gap...}，{1,1+gap,1+2\*gap...}
        2. 每个序列内部使用简单插入排序
        3. 降低gap大小直至1
        4. 优秀的增量序列选择，如hibbard增量，$\\{1,3,7,15...\\},h_{i+1}=2h_{i}+1$，经复杂证明，最坏时间复杂度也低于$O(n^2)$【未确认】
- ## 归并排序
    1. 步骤：
        1. 二分区间，递归直到当前区间只有少量元素，完成小区间上的排序（具体实现任意）
        2. 合并左右两个有序区间，需要额外空间
        3. 重复直到整个区间合并完成
    2. 额外意义：
        1. 归并排序的思想会用在一些区间类型的动态规划问题里。比如求逆序对儿（$x_i > x_j$且$i< j$）的个数，左右区间合并的时候，可以额外计算很多有意义的信息。
- ## 堆排序
    1. 名词解释：
        1. 堆（heap）最早是在堆排序中提出的，后来扩展了代表一类内存区域的含义。注意区分。
        2. （二叉）堆是一种完全二叉树，根据使用场景分为大顶堆和小顶堆。大顶堆中父节点是子树中最大的。小顶堆反之。
        3. 堆也用于实现优先队列。
    2. 步骤：
        1. 建立初始堆。
        2. 循环移动堆顶到序列尾端，并恢复堆性质。
    ```cpp
    // 堆化
    void heapify(vector<int>& input, size_t i, size_t count) {
        while (i<count)
        {
            if (i * 2 + 2 < count && input[i * 2 + 2] == max(max(input[i * 2 + 2], input[i * 2 + 1]), input[i])) { // right child bigger
                swap(input[i * 2 + 2], input[i]);
                i = i * 2 + 2;
            }
            else if (i * 2 + 1 < count && input[i * 2 + 1] == max(input[i * 2 + 1], input[i])) { // left child bigger
                swap(input[i * 2 + 1], input[i]);
                i = i * 2 + 1;
            }
            else {
                break;
            }
        }
    }

    void heap_sort(vector<int>& input) {
	    for (size_t i = (input.size() - 1) / 2; i > 0; i--)
            heapify(input, i, input.size());
        heapify(input, 0, input.size());
        for (size_t i = 1; i != input.size(); ++i) {
            swap(input[0], input[input.size() - i]);
            heapify(input, 0, input.size() - i);
        }
    }
    ```
    3. 多bb一句，计算一下初始化建堆的时间复杂度。
        1. 单次heapify是$O(logn)$，调用了$O(n)$，是$O(nlogn)$吗？**不是!**
        2. 第k层节点（根为第0层）在进行heapify时，最多进行$\lceil logn \rceil -(k+1)$次向下交换。
        3. 第k层节点数量为$2^k$个
        4. 交换次数基本等价于时间复杂度，总交换次数结合2和3可知，等差×等比。结果是$O(n)$
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
    1. 一种最初用在老式穿孔卡上的算法。从分解数字为不同位，并按照每一位进行排序的思路。
    2. 步骤：
        1. 从最低位开始，以最低位为排序元素，对数组元素排序。（一般采用计数排序等稳定排序）
        2. 循环直到取到最高位
    3. 关于表格中的复杂度，d代表位数，r代表每一位的取值范围大小，n为待排序规模。
    4. 多说一句：关于2.1种采用稳定排序的必要性，排序稳定性决定了基数排序的正确性（比如在某位相等时，只有排序顺序稳定则最终结果才一定是正确的）
- ## 桶排序
    1. 概况：当输入符合均匀分布时，可以在线性**期望**时间复杂度下运行。桶就是一系列有大小顺序的区间，算法的实现效率关键在于桶的设计。（准确的说，只要各个桶的元素数量的平方和与总元素数成线性关系即可）
    2. 步骤：
        1. 将数据按桶分配到不同的桶中
        2. 每个桶内部排序（例如插入排序）
        3. 将各个桶直接顺序相连
    3. 为什么使用插入排序还可以是线性**期望**时间复杂度？
        1. 具体证明可查看算法导论相关章节
        2. 核心思路：
            - 总体上是证明$T(n)=\Theta(n)+\sum_{i=0}^{n-1}O(n_{i}^2)$的期望为线性。其中$n_{i}$代表第i个桶中的元素个数。
            - 关键在于证明$\sum_{i=0}^{n-1}O(E[n_{i}^2])$是线性的。（E代表期望）
            - 推理的关键在于令$n_i=\sum_{j=1}^{n}X_{ij}$，其中$X_{ij}=I\\{A[j]$落在桶$i$中$\\}$。$X_{ij}$是指示器随机变量（参考算法导论其一：分析）。
- ## 计数排序
    1. 概况：假设n个输入元素中的每一个都是介于0到k之间的整数，此处k为某个证书。当$k=O(n)$时，计数排序的运行时间是$\Theta(n)$。
    2. 基本思路是对于每一个输入元素x，计数小于x的元素个数。从而直接将x放在指定的位置。
    ```cpp
    std::vector<int> count_sort(std::vector<int>& input) {
        std::vector<int> ret(input.size(), 0);
        auto minmaxElement = std::minmax_element(input.begin(), input.end());
        std::vector<unsigned int> count(*minmaxElement.second - *minmaxElement.first + 1, 0);
        for (auto& t : input) {
            count[t - *minmaxElement.first]++;
        }
        for (size_t i = 1; i != count.size(); ++i) {
            count[i] += count[i - 1];
        }
        for (auto& t : input) {
            cout << count[t - *minmaxElement.first] - 1 << endl;
            ret[count[t - *minmaxElement.first] - 1] = t;
            count[t - *minmaxElement.first]--;
        }
        return ret;
    }
    ```
# 多说几句
1. 基于比较的排序，最优的渐进时间复杂度就是$O(nlogn)$，这一点是有证明的（决策树模型）。