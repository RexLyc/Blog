---
title: "算法导论其十：进阶篇"
date: 2023-12-06T11:04:53+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algorithm.jpg
math: true
---
本章节将会聚焦一些经典问题，以及经典进阶算法。在追求高效率的路上是没有极限的。
<!--more-->
## 经典问题
### RMQ区间最大值最小值查询
问题描述：在一个给定区间中，查询区间内的最大值或最小值
- Tarjan的Sparse-Table算法
    - 构建：$dp[i][j]$表示i为起点，长度为$2^j$的区间的最值。状态转移方程（可以理解为二分）为$dp[i][j]=min/max(dp[i][j-1]+dp[i+(1<<(j-1))][j-1])$。注意二层循环，j为外层（按照区间长度分治）。
    - 查询：寻找小于等于区间范围的区间（或多个区间，可以重叠）。
    - 复杂度：$O(nlogn)$建立，$O(1)$查询
- 线段树：$O(n)$建立，$O(logn)$查询
- 笛卡尔树+LCA+DP：$O(n)$建立，$O(1)$查询

### LCA最近公共祖先
问题描述：在一棵树中，求任意两个节点的祖先中，深度最大的那个。LCA问题往往会同时输入多个查询。
- 朴素算法：
    - 描述：将两个节点中深度较大的一个先向树根上跳，直到二者深度一致。然后再共同上跳，直到相遇。
    - 复杂度：$O(H)$，考虑到$m$次查询，则整体复杂度为$O(H*m)$，$H$为树的高度。
- 倍增算法：
    - 描述：建立一个跳转表，加速朴素算法两个阶段的上跳过程。
        - 跳转表：和RMQ问题中的ST表非常类似，令$f[i][j]$代表点$i$的第$2^j$个祖先，则$f[i][j]=f[f[i][j-1]][j-1]$。（爸爸的爸爸是爷爷）
        - 建立跳转表：dfs遍历，每个节点利用父节点已经保存的祖先信息计算。另外在dfs时顺便保存每个节点的深度信息。
        - 第一阶段上跳：每一次上跳尽量多且不超过另一个节点的深度
        - 第二阶段上跳：两节点上跳到父节点相等，但当前节点不相等。如果上跳过程中两个节点跳后相等，则是跳多了。
    - 复杂度：预处理$O(nlog n)$，单次查询为$O(log n)$，整体为$O(n log n + m log n)$
- 欧拉序列转RMQ问题：
    - 描述：将树转换为欧拉序列，并保存对应的深度信息，再对序列求RMQ问题
        - 欧拉序列：按照dfs顺序，当前节点开始访问和回溯时都记录到序列中，“根-左子树-根-右子树-根”，序列长度为$2n-1$。
        - 欧拉序列性质：树上的两点，其最终在欧拉序列中的两个位置（只取节点第一次加入的位置），两个位置之间，深度最小值就是其LCA
    - 复杂度：由所使用的RMQ算法决定
    - 下面展示一段代码，同时使用了欧拉序列（dfs内）和Tarjan-ST算法，求解一个基于LCA的问题。[边权重均等查询](https://leetcode.cn/problems/minimum-edge-weight-equilibrium-queries-in-a-tree/description/)。
        ```cpp
        class Solution {
            public:

                int calcBit(int n){
                    // 计算二进制下n所需的位数
                    for(int i=1;i<=n;++i){
                        if((1<<i)>n){
                            return i;
                        }
                    }
                    return -1;
                }

                // n为节点数
                // edges为若干三元组，三元组内分别是起点、终点、权重
                // queires是若干查询二元组，每一个内是起点、终点
                vector<int> minOperationsQueries(int n, vector<vector<int>>& edges, vector<vector<int>>& queries) {
                    // 树上路径，统计路径上最多的等权的边。注意该统计满足结合律（当然也满足交换律）
                    // 静态路径，转欧拉序列用倍增法RMQ实现起来比较方便（好写）
                    // 预处理路径转为邻接表
                    unordered_map<int,unordered_map<int,int>> adj;
                    for(auto&t:edges){
                        adj[t[0]][t[1]]=t[2];
                        adj[t[1]][t[0]]=t[2];
                    }
                    
                    vector<int> depthMap(n,-1);
                    vector<int> euler;
                    vector<int> first(n,0);
                    euler.reserve(2*n);
                    // 计算各种权重边的数目
                    vector<vector<int>> dist(n,vector<int>(27,0));
                    // dfs转RMQ
                    function<void(int,int,int)> dfs = [&](int node,int parent,int depth){
                        // 记录第一次加入欧拉序列的位置
                        first[node]=euler.size();
                        // 更新欧拉序列
                        euler.push_back(node);
                        // 更新深度
                        depthMap[node]=depth;
                        // 更新边数
                        if(parent!=-1){
                            dist[node]=dist[parent];
                            dist[node][adj[parent][node]]++;
                        }
                        // dfs
                        for(auto&next:adj[node]){
                            // weight(t.first -> next->first) = next.second;
                            if(depthMap[next.first]==-1){
                                dfs(next.first,node,depth+1);
                                // 更新欧拉序列
                                euler.push_back(node);
                            }
                        }
                    };
                    dfs(0,-1,0);
                    
                    // RMQ预处理，计算全部区间上的最小depth
                    int rangeLength=euler.size();
                    vector<vector<pair<int,int>>> dp(rangeLength,vector<pair<int,int>>(calcBit(rangeLength),make_pair(INT_MAX,0)));
                    // 倍增法
                    for(int i=0;i!=rangeLength;++i){
                        dp[i][0]=make_pair(depthMap[euler[i]],euler[i]);
                    }
                    for(int i=1;i!=calcBit(rangeLength);++i){
                        for(int j=0;j+(1<<i)<=rangeLength;++j){
                            // 核心转移方程
                            dp[j][i]=min(dp[j][i-1],dp[j+(1<<(i-1))][i-1]);
                        }
                    }
                    vector<int> result;
                    
                    auto query = [&](int L,int R){
                        if(L>R)
                            swap(L,R);
                        int i=0;
                        // 找到第一个不超过当前距离的2的整数次幂长度
                        while((1<<(i+1))<=(R-L))
                            ++i;
                        // 由两段拼成
                        return min(dp[L][i],dp[max(L,R-(1<<(i)))][i]).second;
                    };

                    auto weightChange = [&](int lca,int L,int R){
                        // 计算L<->R的边总和，并减去最多的相同同权边
                        vector<int> temp(27,0);
                        int sum=0;
                        int maxCountWeight=0;
                        for(int i=0;i!=27;++i){
                            temp[i]=dist[L][i]+dist[R][i]-2*dist[lca][i];
                            sum+=temp[i];
                            if(temp[i]>temp[maxCountWeight]){
                                maxCountWeight=i;
                            }
                        }
                        return sum-temp[maxCountWeight];
                    };
                    for(auto&t:queries){
                        int lca = query(first[t[0]],first[t[1]]);
                        result.push_back(weightChange(lca,t[0],t[1]));
                    }
                    return result;
                }
            };
        ```
- Tarjan算法：
    - 描述：考虑一个节点的左右子树，如果一个查询分别位于两个子树上，则LCA一定就是当前的节点。如果用并查集来合并保存节点之间的祖先关系，那么这种查询就能在$O(1)$时间内完成。
        - 建立初始并查集（孤立点），并为每一个节点记录所有相关查询
        - 遍历：从根开始DFS遍历每一个节点
            - 检查当前节点是否有查询请求
                - 有，且另一个节点标记为已处理，则另一个节点的并查集祖先就是当前查询的LCA
                - 有，且另一个节点未处理，暂时跳过
            - 如果存在子节点未访问，对其递归
                - 任意子节点递归返回时，都将其并查集合并到当前节点，即以当前节点为子节点的并查集祖先
            - 不存在子节点，直接返回
    - 复杂度：建立数据结构花费$O(n)$，$n$为节点数，单次查询为$O(1)$，整体为$O(n+m)$。
    - 扩展：
        - 求树上两点路径：Tarjan算法在使用的过程中，一个查询中，已处理节点的并查集路径和当前节点的各级父节点就是两点之间的路径
- 扩展
    - LCA和树差分：求一个带权树上两点之间的距离。其距离等于两点到根的距离之和，减去两倍的两点LCA到根的距离。
- 参考：[最近公共祖先 LCA 算法详解- 朴素、在线、离线](https://blog.csdn.net/qq_43332980/article/details/107437070)、[Bilibili推荐：【算法】LCA&RMQ&树差分——保姆级教程](https://www.bilibili.com/video/BV1nE411L7rz)

### 图论-求桥-求割点
主要是针对一个无向图，如果删除当前边后无向图的连通分量增加，则当前边为桥。桥两边的点就是割点。反之不成立，有割点不一定有桥，但是有桥一定有割点。

- 朴素算法（基准法）：
    - 思路：计算连通子图的个数A。删除某个边，并重新计算联通子图的数量B，如果不相等，则该边为桥。稠密图上，复杂度最坏$O(n^4)$。
    - 优化：删除某个边之后，从改边的某一个点开始DFS，如果发现不能到达另一个点，或者已经到达，分别代表是桥，不是桥。
    - 并查集优化：将同一个联通子图用一个并查集表示，此时只需要对所有的边进行遍历（合并并查集），不需要DFS。并重复朴素法，统计连通子图（这里是并查集个数）的思路。
- Tarjan（j发y的音）：参考[Tarjan算法：求解图的割点与桥（割边）](https://www.cnblogs.com/nullzx/p/7968110.html)、代码参考[洛谷 Tarjan 求割点](https://www.luogu.com/article/e4r4ubbt)
    - 思路：Tarjan算法将求桥和求LCA进行了合并。简述一下思路：考虑一次DFS的递归过程。任选一个根，如果我们按照递归的深度去建立一棵树。一个点是割点的条件，就是它至少有一个子结点，只能通过该点连在树上。
        - 特殊点：根，判断根是不是割点，可以通过一次DFS（这里的一次是只选定某一个子节点开始DFS，第一次返回到根节点的时候），能否遍历所有节点来判断。如果不能的话，说明根也是割点。
    - 实现细节：建立三个数组
        - dfn[]：下标是顶点，存储DFS的遍历顺序号，一般可以递增+1来计算，子节点的值一定比父节点的值大
        - low[]：下标是顶点，存储在DFS遍历过程中，不通过父节点，可以访问到的最小的dfn数值。初始值就是dfn。low的计算是最复杂的，但其实也还好，有两种情况
            - 有父子关系的u、v：当一次从节点u(父)到节点v(子)的dfs回溯时。进行更新$low[u]=min(low[u],low[v])$。
            - 没有父子关系的u、v：这种情况出现在dfs扩展时，会发现某一个点v，已经有dfn值，说明已经遍历过，此时虽然没有父子关系，但是由于连接，所以仍需要更新$low[u]=min(low[u],dfn[v])$
        - parent[]：下标是顶点，表示该顶点的父节点，用于在计算low时排除父节点
    - 结果判定
        - 割点：判断顶点U是否为割点，用U顶点的dfn值和它的所有的孩子顶点的low值进行比较，如果存在至少一个孩子顶点V满足low[v] >= dfn[u]，就说明顶点V访问顶点U的祖先顶点，必须通过顶点U，而不存在顶点V到顶点U祖先顶点的其它路径，所以顶点U就是一个割点。对于没有孩子顶点的顶点，显然不会是割点。
            - 另外对于根，单独处理。
        - 桥（割边）：low[v] > dfn[u] 就说明V-U是桥
    > Tarjan的优势是，对于一个连通图，只需要一次dfs，就能找出所有桥和割点。