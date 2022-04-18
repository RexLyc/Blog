---
title: "算法导论其九：实用篇"
date: 2022-03-29T10:23:48+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
本文是以实用角度出发，总结一些比赛和生产中常用的优秀算法和技巧。和算法导论关系不大了。
<!--more-->
## 线段树
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
1. 示例代码：
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

1. 参考
    1. [算法学习笔记(14): 线段树](https://zhuanlan.zhihu.com/p/106118909)
## To Be Continue