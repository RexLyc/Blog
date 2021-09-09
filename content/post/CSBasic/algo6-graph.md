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
		- 实际上，也可以用白色、灰色、黑色，分别代表顶点的未访问，正在扩展后裔、完成扩展这三种状态。
	- 括号定理：在深度优先搜索中，顶点u和v，下述条件有且仅有一个为真
		1. 区间[d[u],f[u]]和区间[d[v],f[v]]完全不相交，且在深度优先森林中，u、v都不是对方的后裔
		2. 区间[d[u],f[u]]$\subset$[d[v],f[v]],且u是v的后裔
		3. 区间[d[u],f[u]]$\supset$[d[v],f[v]],且v是u的后裔
	- 白色路径定理：在图$G=(V,E)$的深度优先森林中，顶点v是顶点u的后裔，当且仅当，在$d[u]$时刻发现u时，可以沿一条全为白色顶点组成的路径到达v。
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
		3. 在逆图上，根据1中的f(u)降序调用dfs。每一次dfs扩展出来的顶点，即每一棵深度优先树，组成一个强连通分支。
	- 方法二（Tarjan算法）：更好更强大的一种求强连通分支的算法。贴一个链接，[史上最清晰的Tarjan算法详解](https://segmentfault.com/a/1190000039149539)。平均比Kosaraju快30%。
	- 方法三（Gabow算法）：Tarjan的提升版本，据说更快一些。[Gabow简单讲解和源码](https://blog.csdn.net/w745241408/article/details/7524523)
- Kosaraju算法的证明：
	- 术语：
		- 逆图$G^{T}=(V,E^{T})$，即对任意$(u,v)\in E^{T}，\mathrm{满足}(v,u)\in E$。
		- 分支图$G^{SCC}=(V^{SCC},E^{SCC})$，scc即strongly connected components，强连通分支。分支图代表了将每一个强连通分支视作一个顶点时，原图收缩后的样子。
		- 扩展发现和完成时间符号d、f定义：$\mathrm{对顶点集}V，d(V)=\min_{v\in V}\\{d(v)\\}，f(V)=\max_{v\in V} \\{f(v)\\}$
	- 引理：
		1. 设$C$和$C'$是有向图$G=(V,E)$中两个不同的强连通分支，设$u,v\in C,u',v'\in C'$，若存在路径$u \leadsto u'$，则不可能同时存在路径$v \leadsto v'$
		2. 设$C$和$C'$是有向图$G=(V,E)$中两个不同的强连通分支，假设有一条边$(u,v)\in E，其中u \in C，v \in C'，则f(C)>f(C')$。
	- 归纳证明：（注意当前的图是逆图$G^{T}$）
		1. 假设头k个深度优先树都是强连通分支。
		2. 第k+1个树的顶点为u，并假设u目前位于强连通分支C中。（接下来证明以u为根的深度优先树恰好等价于强连通分支C）
		3. 根据归纳假设，和白色路径定理，此时C中所有顶点均为u的后裔。
		4. 由于选取顶点由完成时间f决定，此时对于剩余的任何强连通分支$C'$，都有$f(u)=f(C)>f(C')$。此时根据归纳假设和引理第2条，在$G^{T}$中任何离开C的边，必定指向已被访问过的强连通分支（在$G$中，该边是进入$C$的，所以该分支的完成时间f必定更晚，所以在$G^{Y}$中此时已经被访问过了）。
		5. 因此，对于C以外的任何其他强连通分支以外的点，都不可能是u的后裔。综合第3点，说明C中的顶点等价于u的后裔。因此在$G^{T}$中，根为u的深度优先树恰好形成了强连通分支C。
```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<unordered_set>
// 双DFS强连通分支，即Kosaraju算法的实现
void visitDFS(const std::vector<std::vector<int>>& adj
	, std::vector<int>& finishOrder, std::unordered_set<int>& visited, int current) {
	for (auto& t : adj[current]) {
		if (visited.find(t) == visited.end()) {
			visited.insert(t);
			visitDFS(adj, finishOrder, visited, t);
		}
	}
	finishOrder.push_back(current);
}

void finishDFS(const std::vector<std::vector<int>>& adj, std::vector<int>& finishOrder) {
	std::unordered_set<int> visited;
	for (int i = 0; i != adj.size(); ++i) {
		if (visited.find(i) == visited.end()) {
			visited.insert(i);
			visitDFS(adj, finishOrder, visited, i);
		}
	}
}

std::vector<std::vector<int>> reverseGraph(const std::vector<std::vector<int>>& adj) {
	std::vector<std::vector<int>> rAdj(adj.size());
	for (int i = 0; i != adj.size(); ++i) {
		for (auto& t : adj[i]) {
			rAdj[t].push_back(i);
		}
	}
	return rAdj;
}

void visitSCC(const std::vector<std::vector<int>>& adj
	, std::vector<std::vector<int>>& sccGraph, std::unordered_set<int>& visited, int current) {
	
	for (auto& t : adj[current]) {
		if (visited.find(t) == visited.end()) {
			visited.insert(t);
			sccGraph.back().push_back(t);
			visitSCC(adj, sccGraph, visited, t);
		}
	}
}

void getSCC(const std::vector<std::vector<int>>& adj
	, std::vector<std::vector<int>>& sccGraph, std::vector<int> finishOrder) {
	std::unordered_set<int> visited;
	for (auto& t : finishOrder) {
		if (visited.find(t) == visited.end()) {
			sccGraph.push_back(std::vector<int>());
			sccGraph.back().push_back(t);
			visited.insert(t);
			visitSCC(adj, sccGraph, visited, t);
		}
	}
}

void sccDFS(const std::vector<std::vector<int>>& adj, std::vector<std::vector<int>>& sccGraph) {
	std::vector<int> finishOrder;
	finishDFS(adj, finishOrder);
	std::reverse(finishOrder.begin(), finishOrder.end());
	auto rAdj = reverseGraph(adj);
	getSCC(rAdj, sccGraph, finishOrder);
}

int main(){
	std::vector<std::vector<int>> adj = {
		{1}
		,{2}
		,{0,3}
		,{4,5}
		,{5}
		,{4}
	};
	std::vector<std::vector<int>> sccGraph;
	sccDFS(adj, sccGraph);
	for (auto& t : sccGraph) {
		for (auto& tt : t) {
			std::cout << tt << " ";
		}
		std::cout << std::endl;
	}
	return 0;
}
```
# 生成树问题
- 术语
- Prim算法
- Kruskal算法
# 最短路径问题
- 术语
## 单源最短路
- Dijkstra算法
- Bellman-Ford算法
## 多源最短路
- Floyd算法
# 最大流问题
- 流网络：一个有向图$G=(V,E)$，其中每条边$(u,v)\in E$均有一个非负容量$c(u,v)\ge 0$。网络中有两个特别的顶点，源点s，和汇点t。为了方便起见，约定$\forall v \in V, s\leadsto v \leadsto t$。
- 流：在流网络$G=(V,E)$中，流是一个实值函数$f:V \times V \to \mathbf{R}$。可以理解为，流是一个流网络中的流量分配情况。需要满足以下性质：（注意流从定义上的定义域就是$V \times V$的）
	1. 容量限制：$\forall u,v \in V, f(u,v) \le c(u,)$
	2. 反对称性：$\forall u,v \in V, f(u,v) = -f(v,u)$
	3. 流守恒性：$\forall u\in V-{s,t}, \sum_{v\in V}f(u,v)=0$
- 多个源点和多个汇点：转化为具有一个超级源点、超级汇点的流网络，其中超级源点到多个源点，以及多个汇点到超级汇点都具有无限容量。
- 最大流：在给定流网络的前提下，求出网络中能接受的最大流量。
- 术语和基本性质：
	- 流的值：定义为$|f|=\sum_{v\in V}f(s,v)$。即从原点出发的总流。注意符号$|f|$，并不代表流的绝对值。
	- 流的和：设f为G的一个流，$f'$是由f导出的G的残留网络$G_f$的一个流。则$f+f'$也是G中的一个流，其值为$|f+f'|=|f|+|f'|$。
	- 对集合运算：$f(X,Y)=\sum_{x\in X}\sum_{y\in Y}f(x,y)$。其中$X ,Y\subset V,$
		- $f(X,X)=0$
		- $f(X,Y)=-f(Y,X)$
		- 若$X \cap Y = \emptyset$，则有$f(X \cup Y,Z)=f(X,Z)+f(Y,Z)$且$f(Z,X \cup Y)=f(Z,X)+f(Z,Y)$
	- 边残留容量：设f为G的一个流，对顶点$u,v\in V$，若在不超过容量$c(u,v)$的前提下，从u到v可以压入额外的网络流量，就是(u,v)的残留容量，记为$c_f(u,v)=c(u,v)-f(u,v)$
	- 残留网络：给定一个流网络$G=(V,E)$和流$f$，由$f$压得的$G$的残留网络是$G_f=(V,E_f)$，其中$E_f=\\{(u,v)\in V \times V:c_f(u,v)>0\\}$
		- 注意残留网络，不仅包含从s到t方向上，未被填满的边，也包括反方向的边（虽然边实际上是单向的，但正向压入了流量$x$，代表反向可以压入$x$，等价于把这条边的流量清空）
	- 增广路径：已知一个流网络G和流f，增广路径p是残留网络$G_f$中从s到t的一条简单路径。
	- 增广路径的残留容量：能够沿着增广路径p上每一条边传输的网络流的最大值，定义为$c_f(p)=min\\{c_f(u,v):(u,v)\mathrm{在}p\mathrm{上}\\}$
		- 在一个流网络G和流f中，设p是$G_f$的一条增广路径。定义流$f_p:V\times V \to \mathbf{R}$。
		</br><center>$f_p(u,v) = \left\\{ \begin{array}{11}
			c_f(p) & \mathrm{如果}(u,v)\in p\\\\
			-c_f(p) & \mathrm{否则}(v,u)\in p\\\\
			0 & \mathrm{否则}
		\end{array} \right.$</center>
		</br>则$f_p$是$G_f$上的一个流，其值为$|f_p|=c_f(p)>0$
		- 在此基础上，定义流$f'=f+f_p$，则$f'$是G的一个流，其值$|f'|=|f|+|f_p|>|f|$。
	- 割：流网络G的割$(S,T)$将V划分为S和$T=V-S$两个部分，且有$s\in S,t \in T$。如果f是流网络的一个流，则穿过割$(S,T)$的净流被定义为$f(S,T)$。割$(S,T)$的容量是$c(S,T)$。一个网络的最小割就是网络中所有割中具有最小容量的割。
		- 引理：对一个流网络G中任意流f来说，其值的上界为G的任意的割的容量。
- 最大流最小割定理：如果f是具有源点s和汇点t的流网络G中的一个流，则下列条件是等价的：
	1. f是G的一个最大流
	2. 残留网络$G_f$不包含增广路径
	3. 对G的某个割$(S,T)$，有$|f|=c(S,T)$
- Ford-Fulkerson方法：
	- 核心思路：通过不断尝试扩展增广路径，最终获得最大流。
	- Edmonds-Karp算法：用广度优先搜索计算增广路径p。时间复杂度$O(VE^2)$。
		1. 用广度优先搜索，逐层计算从源点s到汇点t的最大流量
		2. 用最大流量更新增广路径上的各边容量，并统计当前流量
		3. 重复步骤1，直到广度优先搜索无法抵达汇点t
- 压入与重标记算法：不检查整个残留网络寻找增广路径，每次仅对一个顶点操作，检查相邻顶点。时间复杂度$O(V^2E)$
	- 前置流：算法使用的一种特殊的流$f:V\times V \to \mathbf{R}$。满足反对称性和容量限制，但**不满足流守恒**，满足$\forall u \in V-\\{s\\},f(V,u)\ge 0$，即除源点以外的顶点的总净流为非负值。进入顶点$u$的总净流也称为进入$u$的余流，记为$e(u)=f(V,u)$。余流大于0称为溢出。
	- 高度函数：流网络G中，设f是G的一个前置流。若函数$h:V\to \mathbf{N}$满足$h(s)=|V|,h(t)=0$且对每条残留边$(u,v)\in E_f$，有$h(u)\le h(v)+1$。则称h为高度函数。
	- 压入操作：如果顶点u是溢出顶点，$c_f(u,v)>0,$且$h(u)=h(v)+1$，则可以应用进行压入操作。将流量$min(e[u],c_f(u,v))$从u的余流中减去，压入(u,v)中，并增加v的余流。
		- 饱和压入：如果压入后$c_f(u,v)=0$，则这一次的压入为饱和压入。否则为不饱和压入。**在一次不饱和压入后，顶点u一定不再溢出**
	- 重标记操作：如果u是溢出顶点，且$\forall (u,v)\in E_f$，有$h[u]\le h[v]$。修改高度函数$h[u]=1+min(h[v]:(u,v)\in E_f)$。注意重标记是针对残留网络进行的。但这个操作并不修改残留网络。
	- 前置流初始化：
	</br><center>$f(u,v) = \left\\{ \begin{array}{11}
			c(u,v) & \mathrm{如果}u=s\\\\
			-c(v,u) & \mathrm{如果}v=s\\\\
			0 & \mathrm{否则}
		\end{array} \right.$ 
	</br>
		$h(u) = \left\\{ \begin{array}{11}
			|V| & \mathrm{如果}u=s\\\\
			0 & \mathrm{否则}
		\end{array} \right.$
	</br>
		$e(v)=c(s,v), e(s)=-\sum c(s,v),{(s,v)\in E}$
		</center>
	- 基本思路：
		1. 初始化前置流
		2. 寻找支持压入或重标记的顶点，重复执行操作
	- 正确性和复杂度的证明内容较多，有兴趣自己看一下吧。
- 重标记与前移算法：时间复杂度$O(V^3)$
	- 压入与重标记算法的一个问题是：算法以任意次序执行压入或重标记操作。实际上这些操作可以被更高效的安排。
	- 容许边：流网络G，前置流f，如果$c_f(u,v)>0$,且$h(u)=h(v)+1$，就说(u,v)是容许边。否则是非容许边。
	- 容许网络为$G{f,h}=(V,E_{f,h})$，其中$E_{f,h}$是容许边的集合。
		- 引理：容许网络不包含回路。
		- 引理：如果顶点u是溢出顶点，且$(u,v)$是容许边，则可以压入，该操作不会产生新的容许边，并且可能将$(u,v)$变为非容许边。
		- 引理：如果顶点u是溢出顶点，且不存在$(u,v)$是容许边，则可以重标记，操作后至少有一条$(u,v)$，即离开u的容许边，但不会有$(v,u)$，即进入u的容许边。
	- 相邻表：重标记和前移算法中的边都存储在相邻表中。如果给定流网络G，对顶点$u\in V-\\{s,t\\}$，相邻表$N[u]$是一个关于G中包含u的残留边中的另一端顶点的单链表。（注意没有$N[s]$或$N[t]$但某点u的相邻表$N[u]$中是可以有$s$和$t$的）
	- 溢出顶点的排除：对于每一个溢出顶点，重复步骤
		1. 从相邻表$N[u]$中取当前溢出顶点的待处理相邻顶点$v$，初始值为链表头。
		2. 如果$v$为空，则重标记当前节点，回到1
		3. 如果$(u,v)$为容许边，流量压入$(u,v)$，转4
			- 这一步可能将流量向$t$有效传输，也可能发现无法传递余流而退回给前驱相邻节点。
		4. 移动待处理顶点为链表的下一个顶点
	- 核心算法：
		1. 初始化前置流
		2. 计算包含$V-\\{s,t\\}$顶点的链表L（L在运算过程中隐含了容许网络的拓扑排序）
		3. 计算$V-\\{s,t\\}$中每个顶点的相邻表
		4. 取L头节点$u$
		5. 若u部位不为空
		6. 记录u的当前高度，在u上调用溢出顶点排除程序
			- 如果之前在u上调用过溢出顶点排除程序，后续调用将在之前访问相邻表$N[u]$的位置继续
		7. 高度不变，赋值$u$为$u$的下一个顶点，转5
		8. 否则（高度一定变高），将$u$移动到L头部，并赋值$u$为$u$的下一个顶点，转5
- 流网络算法应用：
	1. 最大二分匹配：
		- 术语：给定一个无向图$G=(V,E)$，一个匹配是边的子集合$M \subset E$，且满足对所有顶点$v \in V$，M中至多有一条边和v关联，被关联的顶点被称为匹配，否则是无匹配的。
		- 目标：最大匹配是最大势的匹配，势即匹配的边的数量$|M|$。
		- 基本思路：变化无向图为有向图$G'=(V',E')$，其中$V'=V\cup\\{s,t\\}$，$E'=\\{(s,u):u\in L\\}$ 
		</br>&emsp;&emsp;$\cup \\{(u,v):u\in L,v \in R,(u,v)\in E\\} $
		</br>&emsp;&emsp;$\cup\\{(v,t):v\in R\\}$
		</br>有向图$G'$的边集$E'$中每条边具有单位容量。此时该有向图的最大流就是原图的最大匹配。
```python
# 节选自wikipedia，已翻译
# 使用Python语言，用邻接矩阵实现，且实际上未建立相邻表，性能相对较差
# https://en.wikipedia.org/wiki/Push%E2%80%93relabel_maximum_flow_algorithm
def relabel_to_front(C, source: int, sink: int) -> int:
    n = len(C)  # 容量邻接矩阵
    F = [[0] * n for _ in range(n)] # 流矩阵
    # 残留容量即为 C[u][v] - F[u][v]

    height = [0] * n  # 定点高度
    excess = [0] * n  # 余流

	# 记录每个顶点当前初立的相邻表中的顶点
	# 对相邻表很粗糙的实现，并未真正实现表，只记录了当前的相邻顶点
    seen   = [0] * n  
    # 构造不包含源点和终点的链表L
    nodelist = [i for i in range(n) if i != source and i != sink]

    def push(u, v): # 压入
        send = min(excess[u], C[u][v] - F[u][v])
        F[u][v] += send
        F[v][u] -= send
        excess[u] -= send
        excess[v] += send

    def relabel(u):
        # 重标记u找到一个最小可压入高度
        min_height = ∞
        for v in xrange(n): # 邻接矩阵的弊端，遍历所有顶点
            if C[u][v] - F[u][v] > 0:
                min_height = min(min_height, height[v])
                height[u] = min_height + 1

    def discharge(u): # 溢出顶点排除
        while excess[u] > 0:
            if seen[u] < n:  # 相邻表仍有后继顶点
                v = seen[u]
                if C[u][v] - F[u][v] > 0 and height[u] > height[v]: # 寻找可压入的相邻顶点
                    push(u, v)
                else:
                    seen[u] += 1
            else:  # 相邻表中无后继，重标记
                relabel(u)
                seen[u] = 0

    height[source] = n  # 初始化源点高度
    excess[source] = ∞  # 初始化源点余流（偷懒）
    for v in range(n): # 前置流初始化
        push(source, v)

    p = 0
    while p < len(nodelist):
        u = nodelist[p]
        old_height = height[u]
        discharge(u)
        if height[u] > old_height:
            nodelist.insert(0, nodelist.pop(p))  # 发生重标记，进行前移
            p = 0  # 从头开始（这里有待商榷，应该可以直接=1了，=0没意义）
        else:
            p += 1

    return sum(F[source])
```