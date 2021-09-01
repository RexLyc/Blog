---
title: "算法导论其六：图算法"
date: 2021-08-16T10:10:21+08:00
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
最炫酷但其实大部分工程师很少使用的图算法闪亮登场，图算法是很多应用场景最常用的算法。另外本期将会回收系列封面，一起进来看看。
<!--more-->
算法导论中所指的图，是由顶点和边组成的逻辑上的图。这样的图是一些现实问题的抽象表示，算法研究者从中总结了一些重要的基础算法，如最短路径问题、最小生成树问题。
<!-- 插入一张图 -->
# 术语
1. 权：
2. 度：当前顶点所属边的总数
    - 入度：当前顶点作为边终点的数量（仅有向图）
    - 出度：当前顶点作为边起点的数量（仅有向图）
3. $G=(V,E)$：表示一个以V为顶点集，E为边集的图。
	- 有向图的边集：$E\subset V\times V$
	- 无向图的边集：无序的顶点对儿
4. 前驱：路径$(u,v)$中，如果顶点$i$在$j$前面访问，则称$i$是$j$的前驱。
5. 后继：和前驱相反，如果顶点$i$在$j$前面访问，则称$j$是$i$的后继。
6. 路径：从顶点u到定点v且长度为k的路径是顶点序列$v_0,v_1,...,v_k$，且$v_0=u$，$v_k=v$，且对任意$i\in[0,k),(v_i,v_{i+1})\in E$。记为$u \leadsto v$
	- 若路径上的各个点均互不相等，则称为简单路径
7. 图算法中的复杂度表示约定：不写$O(|V|)$，直接写$O(V)$。
8. 完全图：指所有点之间都有边的图。
9. 连通图：指任意点对儿之间都有一条路径的图。
# 数据结构
1. 邻接矩阵：对于一个无权图$G=(V,E)$（有权图同理），维护一个矩阵$A=(a_{ij})$，其中
</br><center>$a_{ij} = \left\\{ \begin{array}{11}
    1 & \mathrm{如果}(i,j)\in E\\\\
    0 & \mathrm{否则}
 \end{array} \right.$</center>
 
2. 邻接表：对于一个无权图$G=(V,E)$（有权图同理），维护一个包含$|V|$个列表的数组$A$组成，该数组的每一个元素$A[u]$是一个列表，列表中的元素$v$，代表$(u,v)\in E$。（A代表Adjacency）
> 邻接矩阵并不一定就浪费空间，如果是无权图，可以用位运算的形式压缩，每个标记只占一个bit，只是这样做确实很麻烦。</br>另外只要是无向图，就可以只用一半的矩阵空间，对于java这种天生支持异形数组的语言来说还是ok的。就是用的时候带着一股邪气。
# 图搜索
1. 广度优先搜索（BFS：breadth-first search）
- 基本思路：算法每一层的搜索，都会沿着已发现的节点的边界，访问所有到当前边界距离为1的所有未发现顶点。原书中证明了广度优先搜索在寻找最短路径方面的正确性。
- 实现思路：广度优先搜索天生适合用队列进行实现，其遍历过程等价于队列节点的入队出队过程。
- 时间复杂度：在不考虑复杂的附加算法的前提下，广度优先搜索的复杂度由源点的**可达**顶点，以及相关边共同构成，为$O(V+E)$。
- 广度优先搜索的性质和术语：
	- 广度优先树：BFS在搜索过程中，实际上建立了一个当前图G的子图，即一个广度优先树。根s即为源点，包含所有可达顶点。**对任意顶点v，从树中s到v的路径即图中s到v的最短距离**。
	- 每个顶点只会被发现一次，除根以外，图G中顶点满足其父节点$\pi[u] \neq \mathrm{NIL}$。
	- 前驱子图（predecessor subgraph）：对图$G=(V,E)$，和给定的源顶点s，其前驱子图$G_\pi=(V_\pi,E_\pi)$，其中$V_\pi=\\{v \in V:\pi[v]\ne \mathrm{NIL}\\} \bigcup \\{s\\}$，$E_\pi=\\{(\pi[v],v):v\in V_pi-\\{s\\}\\}$。
	- 如果$V_\pi$由s可达顶点组成，则前驱子图即为G的广度优先树。实际上在经过一次广度优先搜索之后，就恰好形成了这样一个树，此时表中除了源点以外没有找到自己前驱的节点，都是不可达顶点。
<!-- <details>
 <summary>折叠</summary>
 详细内容
</details> -->

```cpp
// 本段代码实现了对一个二维地图从左上角到右下角的广度优先搜索过程
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_set>
std::pair<std::size_t, std::size_t> operator+(const std::pair<std::size_t, std::size_t>& a
        , const std::pair<std::size_t, std::size_t>& b) {
	std::pair<std::size_t, std::size_t> ret = a;
	ret.first += b.first;
	ret.second += b.second;
	return ret;
}

struct posHasher {
	std::size_t operator()(const std::pair<std::size_t, std::size_t>& value) const{
		return std::hash<std::size_t>()(value.first) ^ std::hash<std::size_t>()(value.second);
	}
};

void bfs(const std::vector<std::vector<int>>& input, std::pair<std::size_t, std::size_t> start) {
	std::queue<std::pair<std::size_t, std::size_t>> bfsQueue;
	std::unordered_set<std::pair<std::size_t, std::size_t>, posHasher> flag;
	flag.insert(start); // 访问标记
	bfsQueue.push(start); // 访问队列
    // 模拟二维地图上只允许向右向下运动
	std::vector<std::pair<std::size_t, std::size_t>> pace = { {0,1},{1,0} }; 
	while (!bfsQueue.empty()) { //仍有节点可访问
		auto point = bfsQueue.front();
		std::cout << input[point.first][point.second] << std::endl;
		bfsQueue.pop();
		for (auto &p:pace) { //尝试扩展邻接节点
			auto nextPoint = point + p;
			if (nextPoint.first<input.size()&&nextPoint.second<input[0].size() // 地图边界
				&&flag.find(nextPoint) == flag.end()) { //未被访问
				flag.insert(nextPoint);
				bfsQueue.push(nextPoint);
			}
		}
	}
}
int main() {
	// 并非邻接矩阵或邻接表，这是一个二维的地图，每一个数字是一个顶点
	std::vector<std::vector<int>> input = {
		{1,2,4,7,11},
		{3,5,8,12,16},
		{6,9,13,17,20},
		{10,14,18,21,23},
		{15,19,22,24,25}
	};
	bfs(input, { 0,0 });
	return 0;
}
```
2. 深度优先搜索
- 基本思路：扩展当前已访问集合时，以特定规则从某未被访问的点开始，沿着一条路径一直扩展顶点，直到无法继续。
- 实现思路：深度优先搜索天生适合用栈实现（不喜欢实现栈直接递归调用即可），扩展节点的过程是入栈，递归返回的过程是出栈。
- 时间复杂度：由于深度优先搜索仍然只访问所有顶点有且只有一次，所以为$\Theta(V+E)$。
- 深度优先搜索的性质和术语：
	- 深度优先搜索森林：一般来说深度优先搜索会访问所有顶点，因此形成的搜索树有多个。
	- 前驱子图：$G_\pi=(V,E_\pi)$，其中$E_\pi=\\{(\pi[v],v):v\in V \mathrm{且} \pi[v]\ne \mathrm{NIL}\\}$。
	- 深度优先森林中边的分类：
		- 树边：深度优先森林$G_\pi$中的边，若顶点v是在探寻边(u,v)时首次被发现，则(u,v)是一条树边。
		- 反向边：连接顶点v到其某一祖先u的边。存在反向边则意味着存在环
		- 正向边：连接顶点u到某个后裔v的非树边。
		- 交叉边：其他类型的边。
	- 深度优先森林中的后裔和祖先：v是u的后裔，当且仅当v是由u扩展出来的子孙节点。此时u是v的祖先。
	- 发现时刻$d[u]$：顶点u在图中被发现并开始访问的时刻（开始扩展u的后裔）
	- 完成时刻$f[u]$：顶点u在图中完成访问的时刻（不会再扩展u的后裔）
	- 括号定理：在深度优先搜索中，顶点u和v，下述条件有且仅有一个为真
		1. 区间[d[u],f[u]]和区间[d[v],f[v]]完全不相交，且在深度优先森林中，u、v都不是对方的后裔
		2. 区间[d[u],f[u]]$\subset$[d[v],f[v]],且u是v的后裔
		3. 区间[d[u],f[u]]$\supset$[d[v],f[v]],且v是u的后裔
```cpp
// 本段代码代表了一个用深度优先搜索从起始地点走到目的地的过程
#include<iostream>
#include<vector>
#include<unordered_set>
#include<stack>
template<typename T>
struct posHasher {
	std::size_t operator()(const std::pair<T, T>& value) const{
		return std::hash<T>()(value.first) ^ std::hash<T>()(value.second);
	}
};

std::pair<int, int> operator+(const std::pair<int, int>& a, const std::pair<int, int>& b) {
	std::pair<int, int> ret = a;
	ret.first += b.first;
	ret.second += b.second;
	return ret;
}

void dfs(const std::vector<std::vector<int>>& input, std::pair<int, int> currentPos
        ,std::pair<int, int > targetPos) {
	std::stack<std::pair<int, int>> pathStack;
	std::unordered_set<std::pair<int, int>, posHasher<int>> flag;
	flag.insert(currentPos);
	pathStack.push(currentPos);
	std::vector<std::pair<int, int>> pace = { {0,1},{-1,0},{0,-1},{1,0} };
	bool find = false;
	bool newPos;
	while (!pathStack.empty()) {
		auto pos = pathStack.top();
		// 扩展标记，如果本轮没有扩展，说明当前节点无法再向下深度搜索
		newPos = false;
		for (auto& p : pace) {
			auto nextPoint = pos + p;
			if (nextPoint == targetPos) {
				pathStack.push(nextPoint);
				find = true; // 结果标记
				break;
			}
			else if (nextPoint.first >= 0 && nextPoint.first < input.size()
				&& nextPoint.second >= 0 && nextPoint.second < input[0].size()
				&& flag.find(nextPoint) == flag.end()
                &&input[nextPoint.first][nextPoint.second]) {
				pathStack.push(nextPoint);
				flag.insert(nextPoint);
				newPos = true;
				break;
			}
		}
		if (find) {
			break;
		}
		else if (!newPos) {
			pathStack.pop();
		}
	}
	if (find) {
		// 输出路径
		std::stack<std::pair<int,int>> result;
		while(!pathStack.empty()) {
			result.push(pathStack.top());
			pathStack.pop();
		}
		while (!result.empty()) {
			std::cout << result.top().first << "," << result.top().second << std::endl;
			result.pop();
		}
	}
	else {
		std::cout << "cant move to target" << std::endl;
	}
}
int main(){
    //表格中1为可走的路径，0为不可走的墙。
	std::vector<std::vector<int>> maze = {
		{1,0,1,1,1},
		{1,1,1,0,1},
		{0,0,1,0,0},
		{1,1,1,1,1},
		{1,0,1,0,1}
	};
	dfs(maze, { 0,0 }, { maze.size() - 1,maze.size() - 1 });
    return 0;
}
```
# 拓扑排序
- 对于有向图而言，图的拓扑是一个非常有用的基础信息。代表了顶点之间的先后顺序
- 输出：一个顶点的线性序列，在该序列中，对任意边$(u,v)\in E$，则必有u在序列中早于v出现。
- 基本思路：
	- 方法一：拓扑排序的顺序，和深度优先搜索森林中任意顶点u的f[u]的升序相同。调用dfs算法，保存完成时间f[u]。
	- 方法二：移除图中入度为0的点及其边，更新子图，重复直到所有顶点被移除。
- 方法一的问题：需要添加额外代码判断是否存在环（方法就是判断是否存在反向边）
```cpp
// BFS算法解决拓扑排序
#include<iostream>
#include<vector>
#include<unordered_set>
#include<stack>
bool topoDfsVisit(const std::vector<std::vector<int>>& adj, std::stack<int>& output
	, std::unordered_set<int>& visited, std::unordered_set<int>& ancient, int current) {
	visited.insert(current);
	ancient.insert(current);//记录祖先
	for (auto& t : adj[current]) {
		if (ancient.find(t) != ancient.end()) {//违反无环要求
			return false;
		}
		else if (visited.find(t) == visited.end()) {//尚未访问
			if (!topoDfsVisit(adj, output, visited, ancient, t)) {//子树违反无环要求
				return false;
			}
		}
	}
	output.push(current);
	ancient.erase(current);//递归返回，从祖先列表中删除自己
	return true;
}

bool topoSortDfs(const std::vector<std::vector<int>>& adj, std::stack<int>& output) {
	std::unordered_set<int> visited;
	std::unordered_set<int> ancient;
	for (int i = 0; i != adj.size(); ++i) {
		if (visited.find(i) == visited.end()) {
			if (!topoDfsVisit(adj, output, visited, ancient, i)) {//子树违反无环要求
				return false;
			}
		}
	}
	return true;
}

int main(){
	// 一个有向无环邻接表
	std::vector<std::vector<int>> adj = {
		{},{2,7},{3},{4,8},{5},{},{7},{3},{5}
	};
	std::stack<int> output;
	if (topoSortDfs(adj, output)) {
		while (!output.empty()) {
			std::cout << output.top() << " ";
			output.pop();
		}
	}
	else {
		std::cout << "no topo, there has circle" << std::endl;
	}
	return 0;
}
```
# 图分解
- 强连通分支：有向图$G=(V,E)$的一个强连通分支就是一个最大的顶点集合，满足$C\subset V$，且对于$\forall u,v \in C, u \leadsto v \mathrm{且} v \leadsto u$。
- 强连通分支下的图分解：将图分为几个强连通分支。通常作为其他复杂算法的基础步骤，先计算每一个分支内的结果。最后合并不同分支之间的结果。
- 基本思路：
	- 方法一（Kosaraju算法）：
		1. 调用一次dfs计算图中各顶点的f(u)
		2. 求原图G的逆图$G^T$
		3. 在逆图上，根据1中的f(u)降序调用dfs。每一次dfs扩展出来的顶点，组成一个强连通分支。
	- 方法二（Tarjan算法）：
	- 方法三（Gabow算法）：
- Kosaraju算法的证明：
	- 术语：
		- 分支图$G^{SCC}=(V^{SCC},E^{SCC})$，scc即strongly connected components，强连通分支。分支图代表了将每一个强连通分支视作一个顶点时，原图收缩后的样子。
		- 扩展发现和完成时间符号d、f定义：$\mathrm{对顶点集}V，d(V)=\min_{v\in V}\\{d(v)\\}，f(V)=\max_{v\in V} \\{f(v)\\}$
	- 引理：
		1. 设$C$和$C'$是有向图$G=(V,E)$中两个不同的强连通分支，设$u,v\in C,u',v'\in C'$，若存在路径$u \leadsto u'$，则不可能同时存在路径$v \leadsto v'$
		2. 
# 生成树问题
# 最短路径问题
# 最大流问题