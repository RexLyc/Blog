---
title: "算法导论其九：实用篇"
date: 2022-03-29T10:23:48+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algorithm.jpg
math: true
---
本文是以实用角度出发，总结一些比赛和生产中常用的优秀算法和技巧。和算法导论关系不大了。
<!--more-->
## 高级数据结构
### 树
#### 线段树
1. 是具有区间查询、区间更新能力的常用基础数据结构。
1. 核心原理：
    1. 每个节点记录：
        1. 当前节点统计的闭区间：$[left,right]$
        1. 左右子节点
        1. 当前节点值（根据题目自行设定）
        1. 当前区间待更新值
    1. 左右子节点划分方法：
        - $mid=\lfloor \frac{left+right}{2} \rfloor$
        - 子树闭区间：$[left,mid]，[mid+1,right]$
    1. 叶子节点的区间左右边界相等
1. 核心操作：
    1. build：递归构建初始线段树
    1. update：递归惰性更新区间，即当有子区间必须单独修改时，才更新当前区间之前积累的更新
    1. push_down：将当前节点区间上未更新的值推送到子节点进行更新
        - 推送的操作原理就是把当前的待更新值，累加到子树的待更新值上
    1. query：区间查询，由于可能对子区间进行查询，因此这一步也可能调用push_down
1. 变种：
    1. 离散化：用于区间可用范围很大，但实际可能出现的数值较少，此时不应直接使用原区间作为线段树的区间定义域。
        1. 方法一：对于离线数据，可以一次性获取所有的$n$个可能取值，并进行排序，将原区间各端点定义域映射到$[0,n]$
        1. 方法二：对于在线数据，可以动态插入。区间定义域仍为原定义域，但最初仅定义全区间，根据数据插入情况，不断增加线段树节点
    2. 静态线段树：树完全不更新，去除待更新值。
    3. 通过数组实现的线段树（完全二叉树）
2. 示例代码：
    - [leetcode-我的日程安排表III](https://leetcode-cn.com/problems/my-calendar-iii/)
        ```cpp
        // 离散、在线数据、动态插入
        class MyCalendarThree {
        public:

            class rangeTreeNode {
            public:
                rangeTreeNode* left, * right;
                int rangeLeft, rangeRight;
                int value;
                int mask;
                rangeTreeNode(int rangeLeft, int rangeRight)
                    :left(nullptr), right(nullptr), value(0)
                    , mask(0), rangeLeft(rangeLeft), rangeRight(rangeRight) {}
            };

            rangeTreeNode* root;

            void push_down(rangeTreeNode* node) {
                if (node->rangeLeft == node->rangeRight)
                    return;
                int mid = (node->rangeLeft + node->rangeRight) / 2;
                if (!node->left)
                    node->left = new rangeTreeNode(node->rangeLeft, mid);
                if (!node->right)
                    node->right = new rangeTreeNode(mid + 1, node->rangeRight);
                // 此处不同题目，计算方式大不相同
                node->left->value += node->mask;
                node->right->value += node->mask;
                node->left->mask += node->mask;
                node->right->mask += node->mask;
                node->mask = 0;
            }

            void update(rangeTreeNode* node, int start, int end, int value) {
                if (node->rangeLeft >= start && node->rangeRight <= end) { // fully included
                    node->value += value;
                    node->mask += value;
                }
                else if (start <= node->rangeRight && node->rangeLeft <= end) {
                    push_down(node);
                    update(node->left, start, end, value);
                    update(node->right, start, end, value);
                    // 此处不同题目，计算方式大不相同
                    node->value = max(node->left->value, node->right->value);
                }
            }

            int queryMax(rangeTreeNode* node, int start, int end) {
                if (node->rangeLeft >= start && node->rangeRight <= end) { // fully included
                    return node->value;
                }
                else if (start <= node->rangeRight && node->rangeLeft <= end) {
                    push_down(node);
                    // 此处不同题目，计算方式大不相同
                    return max(queryMax(node->left, start, end)
                            , queryMax(node->right, start, end));
                }
                else {
                    return -1;
                }
            }

            MyCalendarThree() {
                // 不进行静态build
                root = new rangeTreeNode(0, 1e9);
            }

            int book(int start, int end) {
                update(root, start, end - 1, 1);
                return queryMax(root, 1, 1e9);
            }
        };
        ```
    - 统计0、1数组内某个区间内1的个数，value和mask的更新方式
        ```cpp
        class rangeTreeNode {
        public:
            rangeTreeNode *left,*right;
            int rangeLeft,rangeRight;
            int value;
            int mask;
            rangeTreeNode(int l,int r)
                :rangeLeft(l),rangeRight(r),value(0)
                ,mask(-1),left(nullptr),right(nullptr) {}
        };
        
        rangeTreeNode* rangeTreeRoot;
        
        void push_down(rangeTreeNode* node){
            if(node->rangeLeft==node->rangeRight)
                return;
            int mid=(node->rangeLeft+node->rangeRight)/2;
            if(!node->left)
                node->left=new rangeTreeNode(node->rangeLeft,mid);
            if(!node->right)
                node->right=new rangeTreeNode(mid+1,node->rangeRight);
            if(node->mask<0)
                return;
            node->left->value=node->mask*(node->left->rangeRight-node->left->rangeLeft+1);
            node->left->mask=node->mask;
            
            node->right->value=node->mask*(node->right->rangeRight-node->right->rangeLeft+1);
            node->right->mask=node->mask;
            
            node->mask=-1;
            
        }
        
        void update(rangeTreeNode *node,int start,int end,int value){
            if(node->rangeLeft>=start&&node->rangeRight<=end){
                node->value=value*(node->rangeRight-node->rangeLeft+1);
                node->mask=value;
            }
            else if(node->rangeRight>=start&&node->rangeLeft<=end){
                push_down(node);
                update(node->left,start,end,value);
                update(node->right,start,end,value);
                node->value=node->left->value+node->right->value;
            }
        }
        ```
3. 参考
    1. [算法学习笔记(14): 线段树](https://zhuanlan.zhihu.com/p/106118909)
    2. [史上最详细的线段树教程](https://zhuanlan.zhihu.com/p/34150142)

#### 主席树

#### 伸展树
1. Splay树

#### 树套树
树套树是指一个树的节点也是一个树的数据结构。当无法用简单的数据形式维护多维度信息时，可以考虑使用树套树。根据所需要维护信息的不同，可以选用不同种类的树，线段树、平衡树等。另外树套树多用于在线算法、动态开点的情况。对于允许离线的问题，应该考虑尽量不要使用此种高级数据结构，提高效率降低编程复杂度。
1. 线段树套线段树
    - 经典例题：[三维偏序](https://www.luogu.com.cn/problem/P3810)
1. 线段树套平衡树（线段树节点是平衡树）
    - 经典例题：[二逼平衡树](https://loj.ac/p/106)
2. 


#### 前缀树
- 一个小技巧是给字符串添加一个终止字符’$’，便于标记字符串的终止。
- 参考：[前缀树(字典树、Trie树)](https://www.cnblogs.com/zhouzhiyao/p/12547142.html) 。

#### 后缀树
- 讲的比较好的几篇：https://www.cnblogs.com/xubenben/p/3484988.html 、 https://www.cnblogs.com/xubenben/p/3486007.html 、 https://blog.csdn.net/aiphis/article/details/48489709 、https://www.cnblogs.com/gaochundong/p/suffix_tree.html。
- 几个关键点:
    - 活跃节点的理解（当前能够重叠的部分的起始位置）
    - 活跃半径（重叠长度）
    - 活跃边（可能没有，或者就是重叠的起始字符
    - 每次的三个规则
- 参考：
    1. [后缀树详解及其具体应用](https://blog.csdn.net/Yuzhiyuxia/article/details/24305683)
    1. [后缀树Ukkonen构造法](https://blog.csdn.net/smbroe/article/details/42362347)
    1. [后缀树系列一:概念以及Ukk实现原理](https://blog.csdn.net/fjsd155/article/details/80211145)

#### 其他树
- 参考：[二叉树最近公共祖先（LCA）详解](https://www.hrwhisper.me/algorithm-lowest-common-ancestor-of-a-binary-tree/)

### 可持久化
[可持久化数据结构](https://quant67.com/post/algorithms/ads/persistent/persistent.html)的目标是对数据结构，以及结构变更过程中的所有修改进行记录。即能够查看到最新版本，所有历史版本。可持久化级别分为：
1. 半持久化：读历史，写最新
2. 全持久化：读历史，写历史
3. 可合并持久化：读写历史，合并历史
4. 函数式持久化：每一次修改都是创建新节点，所有历史节点都是只读
这里以半持久化为例，展示对各种基础数据类型进行不同程度的改造


### 数组

#### 后缀数组SA
- 后缀数组（Suffix Array）。实现起来比较好理解，而且速度也不慢（不过一般需要引入额外的信息来解题）。参考[博客](https://www.cnblogs.com/jianglangcaijin/p/6035937.html)、[后缀数组详解](https://zhuanlan.zhihu.com/p/561024497)。DC3算法是比较好的实现方式（比较严格的O(3n)），对基数排序的理解。尤其是字符串的特点，导致划分完的2各部分，内部一定是有序的（长度都不一样，不可能出现完全相等）。而网上的对比可以看一下，起始倍增法也算够优秀（实际运行速度）。一个完整的介绍在[后缀数组简介](https://oi-wiki.org/string/sa/)。
- 相关的术语和数据结构
    - 第i个后缀/后缀i：以第i个字符为起点的后缀。可表示成$S[i]$，S代表原始字符串的后缀。
    - $rank[i]$：后缀i的排序排名
    - $sa[j]$：在$rank$中排名为$j$的后缀，在原串中的起始位置。有$sa[rank[i]]=i$。
    - $height[j]$：后缀排名中，排名为i的后缀和前驱的最长公共前缀。即$LCP(S[sa[j]],S[sa[j-1]])$。后缀数组最重要的**导出数据**。有两条性质。
        - 后缀i和后缀j（假定$i<j$）：最长公共前缀长度为$min \lbrace height[k] , rank[i]<k\ge rank[j] \rbrace$。这条性质比较明显，两个字符串的最长公共前缀，是排序后所有在其中间的后缀之间的最长公共前缀长度的最小值。
        - $h[i]$（辅助证明用）：后缀i和排序在其紧前的后缀的最长公共前缀。即$h[i]=height[rank[i]]$，或者$LCP(S[sa[rank[i]-1]],S[sa[rank[i]]])$。必有$h[i+1]\ge h[i]-1$。因此可以利用这一单调性，加快对$height$的求解。
- SA-IS算法（Induced Sort诱导排序）
    - 思路：
    - 代码
        ```cpp
        // To Do
        ```
- 参考：[后缀数组解析及应用](https://blog.csdn.net/yxuanwkeith/article/details/50636898?_=_)



#### 树状数组
- 参考：[树状数组详解](https://www.cnblogs.com/xenny/p/9739600.html)

## 经典问题
### RMQ区间最大值最小值查询
- Tarjan的Sparse-Table算法
    - 构建：$dp[i][j]$表示i为起点，长度为$2^j$的区间的最值。状态转移方程（可以理解为二分）为$dp[i][j]=min/max(dp[i][j-1]+dp[i+(1<<(j-1))][j-1])$。注意二层循环，j为外层（按照区间长度分治）。
    - 查询：寻找小于等于区间范围的区间（或多个区间，可以重叠）。
- 线段树：$O(n)$建立，$O(logn)$查询
- 笛卡尔树+LCA+DP：$O(n)$建立，$O(1)$查询


## 后端
### 流量限制算法
1. 漏桶：以绝对固定的速率接受请求并进行处理，像是一个桶以固定的速率漏水一样
1. 令牌桶：以固定的速率向桶内放令牌，拿到令牌的请求可以进行处理，因此能够接受一定量的大并发
1. 参考：
    - [高并发系统限流-漏桶算法和令牌桶算法](https://www.cnblogs.com/xuwc/p/9123078.html)