---
title: "算法导论其九：进阶数据结构篇"
date: 2022-03-29T10:23:48+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algorithm.jpg
math: true
---
本文是以实用角度出发，总结一些比赛和生产中常用的优秀算法和技巧。和算法导论关系不大了。
<!--more-->
## 树
### 线段树
1. 是具有区间查询、区间更新能力的常用基础数据结构。
    > 做题别太呆，有些场合利用前缀和、差分数组等简单的数据结构就能解决。不一定所有的区间更新都得用线段树。而且有些场景下，线段树难以应用。比如二维及以上的数据。用二维前缀和、二维差分仍然很方便。但是线段树如果树套树就显然开始离谱了。

3. 核心原理：
    1. 每个节点记录：
        1. 当前节点统计的闭区间：$[left,right]$
        2. 左右子节点
        3. 当前节点值（根据题目自行设定）
        4. 当前区间待更新值
    2. 左右子节点划分方法：
        - $mid=\lfloor \frac{left+right}{2} \rfloor$
        - 子树闭区间：$[left,mid]，[mid+1,right]$
    3. 叶子节点的区间左右边界相等
4. 核心操作：
    1. build：递归构建初始线段树
    2. update：递归惰性更新区间，即当有子区间必须单独修改时，才更新当前区间之前积累的更新
    3. push_down：将当前节点区间上未更新的值推送到子节点进行更新
        - 推送的操作原理就是把当前的待更新值，累加到子树的待更新值上
    4. query：区间查询，由于可能对子区间进行查询，因此这一步也可能调用push_down
5. 变种：
    1. 离散化：用于区间可用范围很大，但实际可能出现的数值较少，此时不应直接使用原区间作为线段树的区间定义域。
        1. 方法一：对于离线数据，可以一次性获取所有的$n$个可能取值，并进行排序，将原区间各端点定义域映射到$[0,n]$
        2. 方法二：对于在线数据，可以动态插入。区间定义域仍为原定义域，但最初仅定义全区间，根据数据插入情况，不断增加线段树节点
    2. 静态线段树：树完全不更新，去除待更新值。
    3. 通过数组实现的线段树（完全二叉树）
6. 示例代码：
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
7. 参考
    1. [算法学习笔记(14): 线段树](https://zhuanlan.zhihu.com/p/106118909)
    2. [史上最详细的线段树教程](https://zhuanlan.zhihu.com/p/34150142)

### 主席树

### 伸展树
1. Splay树

### 树套树
树套树是指一个树的节点也是一个树的数据结构。当无法用简单的数据形式维护多维度信息时，可以考虑使用树套树。根据所需要维护信息的不同，可以选用不同种类的树，线段树、平衡树等。另外树套树多用于在线算法、动态开点的情况。对于允许离线的问题，应该考虑尽量不要使用此种高级数据结构，提高效率降低编程复杂度。
1. 线段树套线段树
    - 经典例题：[三维偏序](https://www.luogu.com.cn/problem/P3810)
1. 线段树套平衡树（线段树节点是平衡树）
    - 经典例题：[二逼平衡树](https://loj.ac/p/106)
2. 


### 前缀树
> 又称Trie树、单词查找树、字典树
- 基本原理：
  - 将一个字符串的每一个位分开，从根开始为每一个字符，建立一个节点，并记录父子关系
  - 对于后续输入的字符串，进行类似操作。显然对于同一个子树下的字符串，他们拥有共同的前缀（从子树的根到整体的根）。因此得名前缀树
  - 一个小技巧是给字符串添加一个终止字符’$’，便于标记字符串的终止。
- 前缀树存在的意义
  - 主要是提高一些特定情况下的字符串匹配速度。一定要记得的一点是，```unordered_set<string>```是**不可能**达到$O(1)$速度的，因此在需要进行大量且有一定重复的匹配时，可以考虑用前缀树优化。
  - ```unordered_set<string>```需要在获取到全部的字符串时才能进行匹配判断，而前缀树可以在输入的过程中就进行判断。如果路径已经不存在，则提前结束匹配。
- 样例代码
    ```cpp
    // 拷贝并修改自OI-WIKI

    struct trie {
        // 根据题目数据范围调整
        int next[100000][26], cnt;
        bool exist[100000];  // 标记是否存在以该结点为结尾的字符串

        // 插入字符串
        void insert(string s) {
            // 从根开始遍历
            int p = 0;
            for (int i = 0; i < s.size(); i++) {
                // 下一个节点
                int c = s[i] - 'a';
                if (!next[p][c]) next[p][c] = ++cnt;  // 如果没有，就添加结点
                    p = next[p][c];
            }
            exist[p] = 1;
        }

        // 从根查找字符串
        bool find(string s) {
            // 从根开始
            int p = 0;
            for (int i = 0; i < s.size(); i++) {
                int c = s[i] - 'a';
                // 不可能存在该字符串
                if (!next[p][c]) 
                    return false;
                // 继续向子树移动
                p = next[p][c];
            }
            return exist[p];
        }

        // 从某个位置开始输入字符，判断是否有结尾
        bool try_move(int &p, char c) {
            // 不存在
            if (!next[p][c])
                return false
            p = next[p][c];
            // 返回是否存在以该字符结束的字符串
            return exist[p];
        }
    };
    ```
- 参考：[前缀树(字典树、Trie树)](https://www.cnblogs.com/zhouzhiyao/p/12547142.html) 。

### 后缀树
- 不再建议学习，可以转去学习后缀数组，实际上可以证明，后缀树的任意算法都可以用增强的后缀数组实现。
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

## 可持久化
[可持久化数据结构](https://quant67.com/post/algorithms/ads/persistent/persistent.html)的目标是对数据结构，以及结构变更过程中的所有修改进行记录。即能够查看到最新版本，所有历史版本。可持久化级别分为：
1. 半持久化：读历史，写最新
2. 全持久化：读历史，写历史
3. 可合并持久化：读写历史，合并历史
4. 函数式持久化：每一次修改都是创建新节点，所有历史节点都是只读
这里以半持久化为例，展示对各种基础数据类型进行不同程度的改造


## 数组

### 后缀数组SA
- 后缀数组（Suffix Array）：实现起来比较好理解，而且速度也不慢（不过一般需要引入额外的信息来解题）
- 此节使用的相关的术语和数据结构
    - $charAt[i]$：第i个字符
    - 第i个后缀/后缀i：以第i个字符为起点的后缀。可表示成$S[i]$，S代表原始字符串的后缀。
    - $rank[i]$：后缀i的排序排名
    - $sa[j]$：在$rank$中排名为$j$的后缀，在原串中的起始位置。有$sa[rank[i]]=i$。
    - $height[j]$：后缀排名中，排名为i的后缀和前驱的最长公共前缀。即$LCP(S[sa[j]],S[sa[j-1]])$。后缀数组最重要的**导出数据**。有两条性质。
        - 后缀i和后缀j（假定$i<j$）：最长公共前缀长度为$min \lbrace height[k] , rank[i]<k\ge rank[j] \rbrace$。这条性质比较明显，两个字符串的最长公共前缀，是排序后所有在其中间的后缀之间的最长公共前缀长度的最小值。
        - $h[i]$（辅助证明用）：后缀i和排序在其紧前的后缀的最长公共前缀。即$h[i]=height[rank[i]]$，或者$LCP(S[sa[rank[i]-1]],S[sa[rank[i]]])$。必有$h[i+1]\ge h[i]-1$。因此可以利用这一单调性，加快对$height$的求解。证明步骤如下（后缀k紧邻后缀i）
        ![h证明](/images/algoSeries/proof-sa-height.png)
        - 有了上述定理之后，可以保证$O(n)$时间内求出$height$
            ```cpp
            // 原始串、各后缀在字典序下的序号、按字典序排序下的后缀编号
            void calculate_height(const string& str
                                , const vector<int>& rank
                                , const vector<int>& sa
                                , vector<int>& height) {
                int n = str.size();
                // 前一个前缀计算的height
                int last_height = 0;
                for(int i = 0; i != n; ++i) {
                    // 获得当前后缀i的排序
                    int current = rank[i];
                    if(current == 0) {
                        // 对于字节序最小的后缀，单独处理
                        height[current] = 0;
                        last_height = 0;
                    } else {
                        // 相当于 h[i]-1
                        if(last_height > 0) --last_height;
                        // 获得字典序下的紧前后缀
                        int before_current = sa[current - 1];
                        while(i + last_height < n 
                            && before_current + last_height < n
                            && str[i + last_height] 
                                == str [before_current + last_height]) {
                            ++last_height;
                        }
                        height[current] = last_height;
                    }
                }
            }
            ```
- 倍增法
  - 较易理解和实现，时间复杂度$O(nlog^2n)$，改用基数排序可以做到$O(nlogn)$
  - 思路
    - 计算初始每个字符的排序，注意相同字符的排序应该相同
    - 每次将上一次排序的两个排序序号结果拼接（距离step），对拼接结果排序，并对step倍增。由此不断得到更长的后缀排序
    - 用排序结果计算sa、rank，并进一步计算height
    - 利用三个数组，完成题目内容
  - 代码：以[最长重复子串（允许重叠）](https://leetcode.cn/problems/longest-duplicate-substring/description/)为例
    ```cpp
    string longestDupSubstring(string s) {
        vector<int> rank(s.size(),0),sa(s.size(),0),height(s.size(),0);
        // 倍增法排序
        vector<pair<int,int>> sum(s.size());
        vector<int> order(s.size(),0);
        int step=1;
        // 首轮排序
        for(int i=0;i!=s.size();++i){
            // 值和前缀下标
            sum[i]=make_pair(s[i]-'a',i);
        }
        sort(sum.begin(),sum.end(),[](auto &a,auto &b){return a.first<b.first;});
        // 整理排序结果
        // 对于相同的数字，不排先后顺序
        int currentRank=1;
        for(int i=0;i!=s.size();++i){
            // 判断是否增加rank
            if(i!=0&&sum[i].first!=sum[i-1].first){
                currentRank++;
            }
            order[sum[i].second]=currentRank;
        }
        // ********** 核心 **********
        while(step<s.size()){
            // 倍增
            for(int i=0;i!=s.size();++i){
                sum[i]=make_pair(order[i]*(s.size()+1)+(i+step>=s.size()?0:order[i+step]),i);
            }
            sort(sum.begin(),sum.end(),[](auto &a,auto &b){return a.first<b.first;});
            // 整理排序结果
            currentRank=1;
            for(int i=0;i!=s.size();++i){
                // 判断是否增加rank
                if(i!=0&&sum[i].first!=sum[i-1].first){
                    currentRank++;
                }
                order[sum[i].second]=currentRank;
            }
            // print(order);
            step<<=1;
        }
        // 用排序结果更新rank、sa
        // 注意前面为了方便计算采用的序号从1开始，这里改回来
        for(int i=0;i!=s.size();++i){
            rank[i]=order[i]-1;
        }
        for(int i=0;i!=s.size();++i){
            sa[rank[i]]=i;
        }
        // 计算height
        calculate_height(s,rank,sa,height);
        // 选择具有最大height的任意一个
        auto index=distance(height.begin(),max_element(height.begin(),height.end()));
        // 注意先获取LCP长度
        int length=height[index];
        // 再将index从字典序的index，换为原串的index
        index=sa[index];
        return s.substr(index,length);
    }
    ```
- DC3
  - 较易实现，常数较大，时间复杂度$O(n)$。主要考验对基数排序的理解。尤其是字符串的特点，可以证明划分完的2个部分，内部一定是绝对有序的（长度不一样，不可能出现相等）
- SA-IS算法（Induced Sort诱导排序）
  - 较难实现，时间复杂度最优，$O(n)$
  - 术语：
    - 填充结束字符\#：填充一个比原串所有字符都小的字符到字符串结尾，作为结束字符。本节以\#为例
    - S型后缀：对后缀i，如果有字典序$S[i]<S[i+1]$，则为S型后缀。填充结束字符默认为S型。
    - L型后缀：对后缀i，如果有字典序$S[i]>S[i+1]$，则为L型后缀
    - 后缀类型递推性质：后缀类型可以从尾向头递推，规则为比较首字符，不等则显然，相等则继承上一个后缀的类型。即
        </br><center>$ Type(S[i]) = \left\\{ \begin{array}{21}
            Type(S[i+1]) & charAt[i] == charAt[i+1] \\\\
            L.Type & charAt[i]>charAt[i+1] \\\\
            S.Type & charAt[i]<charAt[i+1]
        \end{array} \right.$
        </center>
    - 后缀类型排序性质：对后缀i和后缀j，如果charAt[i]=charAt[j]，且后缀i为S型，后缀j为L型，则必有字典序后缀i>后缀j。
        - 证明：考虑到S、L型条件，显然一定存在某一位字符，满足后缀i>后缀j
    - \*型后缀：一种特殊的S型后缀，要求其左侧紧前后缀（比它长1个字符的）是L型的。LMS（Left Most S-type）就表达了这个含义。\*型后缀起始位置的字符称为LMS字符，LMS子串是每两个最近的LMS字符构成的子串（包括两个LMS字符）。有如下性质
        - 特殊的，\#是最短的LMS子串
        - 两个LMS字符中间至少有一个L型后缀。因此其他LMS子串长度都必大于2。
        - LMS子串的数量不超过原字符串长度的一半
        - 对任意两个LMS子串，不存在其中一个是另一个的真前缀。这一点可联系\*型后缀的定义来证明。
        - 所有LMS子串的长度之和为$O(原字符串长度)$
          - 因此利用基数排序，可以在$O(n)$时间内，完成对所有LMS子串的排序，注意相等子串排序相等
          - 问题缩减：对各LMS子串，在排序中的序号，排列后做重新命名（一般就直接用序号数字），形成一个新的字符串$S'$，例如排序顺序为1120，则$S'=<1,1,2,0>$。这个新字符串的后缀间的字典序，等价于对应的\*型后缀的字典序。
            - 证明：
              1. 考虑对$S'$中两个字符的比较，由定义可知就是在比较LMS子串，而LMS子串要么完全相等，要么在某一位上有区别。
              2. 因此对于LMS子串完全相等的情况，此时$S'$中对应两个字符所代表的$S'$的后缀的比较，需要递归的去比较后缀的剩余部分。显然这和对应的\*后缀的比较结果是一样的。
              3. 如果LMS子串不相等，则显然$S'$中两个字符的比较，就已经代表了\*后缀的比较结果。甚至连剩余部分的后缀都不需要考虑。
    - $S'$：从\*后缀一步一步构造的代表LMS子串排序的字符串。
        - 计算$S'$对应的后缀数组，得出$SA'$，为诱导排序准备
    - 诱导排序：
        - 诱导排序是一个归纳递推的过程
        - 思路1：显然可以将后缀按首字母分桶，而且在一个桶内的后缀，在最终排序后一定呈现L型后缀在前（字典序小）、S型后缀在后的特点（参考后缀类型排序性质）。因此只需要分别为L、S型后缀排序即可。可将其分别称为L桶、S桶。每个首字母桶下，都分别有一个L、S桶，不过有可能有的L、S桶是空的。
        - 思路2：假定已知\*后缀的大小顺序，则
            - 先用\*后缀，按思路1中的方式分桶，在每个S桶内建立一个初始有序的后缀序列。注意这一步的顺序并不完全正确。\*后缀作为S型后缀的一种，其并不代表全部S型后缀的最终正确顺序。
            - 导出L后缀：置各桶指针为桶首，按字典序从小到大取出集合中各桶内的每一个后缀，如果其紧前后缀是一个L型后缀，将其加入集合中的对应桶（这一步称为导出）。有两条性质可归纳证明。
                - 同一个桶内，L型后缀加入顺序一定严格满足字典序：同一个桶内的L型后缀的首字母相等，其字典序依赖于后缀的剩余部分，而这部分的顺序是已经被导出的，其字典序已知。
                - 一次遍历后所有的L型后缀都将加入到集合中：S型导出L型时，显然导出的L型大于当前S型后缀；L型导出L型时，同为L型后缀的后缀i和后缀j，如果后缀i<后缀j，则后缀i一定先于后缀j加入集合。
            - 再导出全部S后缀：目前L型后缀的顺序已经完全是正确的。重置各桶指针为桶尾，从集合的各桶内，倒字典序取出每一个后缀，其紧前后缀如果是S型，则将其加入（覆盖）到集合的S桶内。仍然可以归纳证明。
                - 同一个桶内，S型后缀加入顺序一定严格满足字典序
                - 所有的S型后缀都将加入到集合中
    - 原始字符串中\*后缀的排序问题。
        - 首先要解决：LMS子串的排序问题。
            - 情况1：LMS子串首字母均不相同，直接对首字母桶排序
            - 情况2：LMS子串首字母存在相同的情况，可以对此时无序的\*后缀进行一次诱导排序，即将\*后缀随意放入每个S桶内。这一步的诱导排序后只能保证此时\*后缀的LMS前缀是有序的。也可以用归纳法证明：
                - 引入术语LMS前缀：$pre(S,i)$，代表后缀i从位置i开始向字符串尾部，直到第一个LMS字符（包括）组成的字符串。对于LMS字符，它的LMS前缀就是其自身。
                - 只需要证明诱导排序，对相同首字母的LMS前缀能有效排序即可
                - 正序遍历导出L型后缀时：可以证明所有L型后缀的LMS前缀都是有序的（单调不递减），反证法，假设加入过程保证有序。
                    - 初始时，只有\*型后缀的LMS前缀，也就是说LMS前缀至多一个字符，就是LMS字符本身。
                    - 当已有k个L型后缀时，加入第k+1个L型后缀，设为后缀i，此时判断该后缀和所在L型桶内的其他L型后缀的LMS前缀的大小。如果此时存在一个L型后缀j大于当前加入的L型后缀i。则必有$pre(S,j+1)>pre(S,i+1)$。但显然包含LMS前缀$pre(S,j+1)$、$pre(S,i+1)$的后缀已经加入到集合中，这些后缀可能是S型也可能是L型，而且根据假设，这些LMS前缀有序，即$pre(S,j+1)<pre(S,i+1)$。矛盾，因此只能$pre(S,j)<pre(S,i)$
                - 倒序遍历重新导出S型后缀：可以证明，此时所有的LMS子串都已完成排序
                    - 在初始化时，LMS子串顺序并不是正确的，只是LMS前缀的顺序正确（此时就是LMS字符本身）
                    - 而倒序重新导出时，最早初始化的LMS字符全部抛弃。此时假设已导出的LMS前缀是有序的。如果发生乱序情况，同样可以证明，其LMS前缀的剩余部分违反了假设。因此导出S型也是有序的。这一步相当于不断在LMS前缀左侧新增S型字符，最终将重新出现各个\*型后缀。
        - LMS子串排序后，根据排序结果，整理序号，对LMS子串做离散化，也就是拿序号替换LMS子串，获得$S'$
        - 如果$S'$内没有重复的字符，则可以直接得出$SA'$并返回
        - 否则，递归调用SA-IS算法，计算$S'$的后缀数组$SA'$，并根据递归结果，计算当前字符串排序结果
    - SA-IS算法整体思路：
        - 输入原字符串$S$
        - 倒序扫描字符串，确定每一个后缀的L、S类型
        - 确定所有的LMS子串
        - 对LMS子串进行诱导排序，并重命名，形成新串$S'$
            - 如果$S'$中每个字符都不一样，直接计算$SA'$
            - 否则递归计算$SA'$
        - 利用$SA'$诱导，对$S$后缀排序，得出$SA$
        - 返回$SA$
    - 代码
        ```cpp
        // to do
        ```
- 参考：[后缀数组解析及应用](https://blog.csdn.net/yxuanwkeith/article/details/50636898?_=_)、参考[后缀数组：倍增法和DC3的简单理解](https://www.cnblogs.com/jianglangcaijin/p/6035937.html)、[后缀数组详解](https://zhuanlan.zhihu.com/p/561024497)、[后缀数组简介](https://oi-wiki.org/string/sa/)、[诱导排序与 SA-IS 算法](https://riteme.site/blog/2016-6-19/sais.html)、[还在写倍增后缀数组? SA-IS算法了解一下~](https://www.luogu.com.cn/blog/ShadowassIIXVIIIIV/on-hou-zhui-shuo-zu-sa-is-suan-fa)、[SA-IS学习笔记 ](https://www.cnblogs.com/Flying2018/p/13848482.html)

### 树状数组
- 树状数组的题目都可以用线段树解决
- 参考：[树状数组详解](https://www.cnblogs.com/xenny/p/9739600.html)

