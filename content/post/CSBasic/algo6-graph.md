---
title: "算法导论其六：图算法"
date: 2021-08-16T10:10:21+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/algorithm.jpg
math: true
---
最炫酷但其实大部分工程师很少使用的图算法闪亮登场，图算法是很多应用场景最常用的算法。另外本期将会回收系列封面，一起进来看看。
<!--more-->
算法导论中所指的图，是由顶点和边组成的逻辑上的图。这样的图是一些现实问题的抽象表示，算法研究者从中总结了一些重要的基础算法，如最短路径问题、最小生成树问题。
<!-- 插入一张图 -->
## 术语
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
## 数据结构
1. 邻接矩阵：对于一个无权图$G=(V,E)$（有权图同理），维护一个矩阵$A=(a_{ij})$，其中
</br><center>$a_{ij} = \left\\{ \begin{array}{11}
    1 & \mathrm{如果}(i,j)\in E\\\\
    0 & \mathrm{否则}
 \end{array} \right.$</center>
 
2. 邻接表：对于一个无权图$G=(V,E)$（有权图同理），维护一个包含$|V|$个列表的数组$A$组成，该数组的每一个元素$A[u]$是一个列表，列表中的元素$v$，代表$(u,v)\in E$。（A代表Adjacency）
> 邻接矩阵并不一定就浪费空间，如果是无权图，可以用位运算的形式压缩，每个标记只占一个bit，只是这样做确实很麻烦。</br>另外只要是无向图，就可以只用一半的矩阵空间，对于java这种天生支持异形数组的语言来说还是ok的。就是用的时候带着一股邪气。
## 图搜索
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
## 拓扑排序
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
## 图分解
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
## 最小生成树问题
- 简单描述：最小生成树问题是一类应用广泛的问题，比如将多个地点连通，或连接多个电位相同的电子元件，在这种情况下，最终生成的边数恰好比顶点数少1，且所有顶点连通。如何寻找满足条件的，总权值和最小的边集合，就是最小生成树问题。
- 术语：
	- 标准定义：一个无向连通图$G=(V,E)$，对图中每一条边$(u,v)\in E$，都有一个权值$w(u,v)$表示连接$u$和$v$的代价。希望找出一个无回路的子集$T\subset E$，它连接了所有的顶点，其权值和$w(T)=\sum_{(u,v)\in T} w(u,v)$最小。由于$T$无回路且连接所有的顶点，它也同时成为一棵树，称为生成树。确定$T$的问题，即为最小生成树问题。
	- 割：无向图$G$的一个割$(S,V-S)$是对$V$的一个划分。
		- 当一条边$(u,v)\in E$的一个端点属于$S$，而另一个端点属于$V-S$，则称边$(u,v)$通过割$(S,V-S)$。
		- 如果一个边集$A$中，没有边通过割$(S,V-S)$，则称该割不妨碍该边集
		- 如果某条边的权值是通过一个割的所有边中最小的，则称该边为通过这个割的一个轻边
	- 安全边：若边$(u,v)$加入一个最小生成树子集A，仍能使$A \cup \\{(u,v)\\}$为最小生成树的子集，则称其为安全边。
		- 引理：设无向图$G=(V,E)$，$A$是$E$的一个最小生成树子集，若割$(S,V-S)$不妨碍$A$，且边$(u,v)$是该割的一个轻边，则该边是集合$A$的安全边。
	- 通用思路：
		1. 初始化最小生成树子集A
		2. 寻找一条安全边，加入子集A
- Kruskal算法
	- 贪心策略：Kruskal算法中，加入集合$A$的安全边，总是连接图中两个不同连通分支的最小权边。过程中$A$可能是一个森林（多棵树）。
	- 基础算法：
		1. 排序所有边
		2. 从小到大，选取各边，若两端在不同连通分支则将该边加入
		3. 循环直到找到$|V|-1$条边
	- 时间复杂度：$O(ElogE)$
- Prim算法
	- 贪心策略：Prim算法中，$A$一直保持单棵树，每一次加入的安全边，总是连接树和一个不在树中的顶点，并且这条安全边是当前可选的安全边中最短的一个。
	- 时间复杂度：$O(ElogV)$，[斐波那契堆]({{<relref "/content/post/CSBasic/algo7-tree.md#斐波那契堆">}})优化下能更快。
		- 注意$O(ElogV)$和$O(ElogE)$并没有本质区别，因为$|E|\le|V|^2$，易知$O(logE)=O(logV)$
	- 优化：
```cpp
// kruskal & prim
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <iostream>
#include <unordered_set>
#include <queue>

typedef unsigned int node_id;

struct edge {
	node_id begin;
	node_id end;
	int weight;

	edge(node_id begin, node_id end, int weight) {
		this->begin = begin;
		this->end = end;
		this->weight = weight;
	}

	bool operator<(const edge& b) {
		return this->weight < b.weight;
	}
};

class union_set {
private:
	std::unordered_map<node_id, node_id> set_;
public:
	void set_parent(node_id child, node_id parent) {
		set_[child] = parent;
	}

	node_id get_parent(node_id child) {
		std::vector<node_id> path;
		node_id current=child;
		while (set_[current] != current) {
			path.push_back(current);
			current = set_[current];
		}
		for (auto& t : path) {
			set_[t] = current;
		}
		return current;
	}

	void make_union(node_id a, node_id b) {
		set_parent(get_parent(a), get_parent(b));
	}

	bool union_check(node_id a, node_id b) {
		return get_parent(a) == get_parent(b);
	}

	std::size_t size() {
		return set_.size();
	}
};

class minimum_spanning_tree {
public:
	virtual int get_minimum_spanning_tree(const std::vector<edge>& graphEdges) = 0;
};

class kruskal : public minimum_spanning_tree {
public:
	int get_minimum_spanning_tree(const std::vector<edge>& graphEdges)
	{
		std::vector<edge> graph(graphEdges.begin(), graphEdges.end());
		std::sort(graph.begin(), graph.end());
		union_set vertexSet;
		for (auto& t : graph) {
			vertexSet.set_parent(t.begin, t.begin);
			vertexSet.set_parent(t.end, t.end);
		}
		std::size_t edge_count = 0;
		int weight_sum = 0;
		for (auto& t : graph) {
			if (!vertexSet.union_check(t.begin, t.end)) {
				edge_count++;
				weight_sum += t.weight;
				vertexSet.make_union(t.begin, t.end);
			}
			if (edge_count == vertexSet.size()) {
				break;
			}
		}
		return weight_sum;
	}
};

struct distance_to_node {
	int weight;
	node_id node;

	distance_to_node(int weight, node_id node) {
		this->weight = weight;
		this->node = node;
	}

	bool operator<(const distance_to_node& a) const {
		return this->weight < a.weight;
	}
};

template<typename T>
struct min_heap {
	bool operator()(const T& a, const T& b) {
		if (!(a < b) && !(b < a)) // strict weak order
			return false;
		else
			return !(a < b);
	}
};

class prim : public minimum_spanning_tree {
public:
	int get_minimum_spanning_tree(const std::vector<edge>& graphEdges)
	{
		std::priority_queue<distance_to_node
			,std::vector<distance_to_node>
			, min_heap<distance_to_node>> distanceQueue;
		std::unordered_map<node_id, std::unordered_map<node_id, int>> adj;
		for (auto& t : graphEdges) {
			adj[t.begin][t.end] = t.weight;
			adj[t.end][t.begin] = t.weight;
		}
		std::unordered_set<node_id> seen;
		distanceQueue.push(distance_to_node(0, graphEdges.front().begin));
		std::size_t edge_count = 0;
		int weight_sum = 0;
		while (!distanceQueue.empty()&&edge_count!=adj.size()) {
			distance_to_node current = distanceQueue.top();
			distanceQueue.pop();
			//std::cout << seen.find(current.node) <<" "<< seen.end() << std::endl;
			if (seen.find(current.node) == seen.end()) {
				seen.insert(current.node);
				edge_count++;
				weight_sum += current.weight;
				for (auto& t : adj[current.node]) {
					if (seen.find(t.first) == seen.end()) {
						distanceQueue.push(distance_to_node(t.second, t.first));
					}
				}
			}
		}
		return weight_sum;
	}
};

int main() {
	minimum_spanning_tree* myTree = new kruskal;
	std::vector<edge> graphEdges;
	graphEdges.push_back(edge(0, 1, 1));
	graphEdges.push_back(edge(1, 4, 1));
	graphEdges.push_back(edge(1, 3, 2));
	graphEdges.push_back(edge(3, 5, 1));
	graphEdges.push_back(edge(2, 3, 1));
	graphEdges.push_back(edge(0, 2, 2));
	graphEdges.push_back(edge(2, 4, 2));
	graphEdges.push_back(edge(4, 5, 3));
	std::cout << myTree->get_minimum_spanning_tree(graphEdges) << std::endl;
	delete myTree;
	myTree = new prim;
	std::cout << myTree->get_minimum_spanning_tree(graphEdges) << std::endl;
	return 0;
}
```
## 最短路径问题
- 术语：
	1. 对于带权有向图$G=(V,E)$，加权函数$w:E\to \mathbf{R}$为从边到实型权值的映射。路径$p=<v_0,v_1,...,v_k>$的权是其组成边的所有权值之和。$w(p)=\sum_{i=1}^{k}w(v_{i-1},v_i)$。定义从$u$到$v$的最短路径的权为：
	</br><center>$\delta(u,v) = \left\\{ \begin{array}{11}
		min\\{w(p):u\leadsto^p v\\} & \mathrm{如果两点间存在路径}\in E\\\\
		\infty & \mathrm{否则}
	\end{array} \right.$</center>
	</br>从顶点$u$到$v$的最短路径定义为权$w(p)=\delta(u,v)$的任何路径。
	2. 单源最短路问题：寻找某个给定的固定的源点$s$到所有顶点$v\in V$的最短路径。
	3. 多源最短路：寻找所有顶点对儿的最短路径。
	3. 最短路径的最优子结构：一条两顶点间最短路径包含路径上其他顶点间的最短路径。
		- 即：最短路径的子路径也是最短路径。
	4. 松弛（单源最短路）技术：松弛一条边$(u,v)$的过程中，要测试是否可以通过$u$，对迄今为止找到的到$v$的最短路径进行优化（更新其最短路径估计$d[v]$）。
		- 三角不等式：$\delta(s,v)\le\delta(s,u)+w(u,v)$
		- 上界性质：最短路径估计$d[v]\ge\delta(s,v)$
		- 无路径性质：若不存在路径，则$d[v]=\delta(s,v)=\infty$
		- 收敛性质：若松弛边$(u,v)$之前的任何时间$d[u]=\delta(s,u)$，则松弛过后总有$d[v]=\delta(s,v)$
		- 路径松弛定理：若$p=<v_0,v_1,...,v_k>$是一条最短路径，且对$p$的边按照$(v_0,v_1)(v_1,v_2)...(v_{k-1},v_k)$进行松弛，则$d[v_k]=\delta(s,v_k)$。
		- 前驱子图性质：一旦对于所有$v\in V,d[v]=\delta(s,v)$，则前驱子图构成一个以s为根的最短路径树。
### 单源最短路
- Dijkstra算法：不存在负权边的情况下，时间复杂度$O(V^2)$，[斐波那契堆]({{<relref "/content/post/CSBasic/algo7-tree.md#斐波那契堆">}})优化下能更快。
	- 基本思路：
		1. 初始化最短路径估计
		2. 初始化已获得最短路径的顶点集合，空集$S$
		3. 初始化以最短路径估计为关键字的顶点小顶堆（最小优先队列），初始只有源点
		4. 穷尽堆顶顶点，直到获得一个未加入$S$的顶点$u$，加入$S$
		5. 松弛$u$的邻接边，将松弛边的相关顶点加入最小优先队列，回到4
	- 正确性证明思路：归纳法验证，一旦将顶点$u$加入$S$时，必有$d[u]=\delta(s,u)$。
	- 正确理解Prim算法和Dijkstra算法的区别。
```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>
#include<unordered_set>

struct edge_adjacent {
	node_id end;
	edge_weight weight;

	edge_adjacent(node_id end, edge_weight weight) {
		this->end = end;
		this->weight = weight;
	}

	bool operator<(const edge_adjacent& b) const {
		return this->weight < b.weight;
	}
};

typedef edge_adjacent distance_single_source;

struct node_adjacent {
	node_id begin;
	std::vector<edge_adjacent> edges;
	node_adjacent(node_id begin, std::vector<edge_adjacent>&& edges) {
		this->begin = begin;
		this->edges = edges;
	}
};

struct graph_adjacent {
	std::vector<node_adjacent> nodes;
};

struct graph_adjacent_map {
	std::unordered_map<node_id, std::vector<edge_adjacent>> nodes;
};

template<typename T>
struct min_heap {
	bool operator()(const T& a, const T& b) {
		if (!(a < b) && !(b < a)) // strict weak order
			return false;
		else
			return !(a < b);
	}
};

std::unordered_map<node_id, edge_weight> dijkstra(const graph_adjacent_map& graph,node_id source) {
	std::unordered_map<node_id, edge_weight> dist;
	std::priority_queue<edge_adjacent, std::vector<edge_adjacent>, min_heap<edge_adjacent>> node_distance;
	std::unordered_set<node_id> arrived_nodes;
	node_distance.push(edge_adjacent(source, 0));
	dist[source] = 0;
	while (!node_distance.empty()) {
		auto node = node_distance.top();
		node_distance.pop();
		if (arrived_nodes.find(node.end) != arrived_nodes.end()) {
			continue;
		}
		// new node
		arrived_nodes.insert(node.end);
		for (auto edge : graph.nodes.find(node.end)->second) {
			if (dist.find(edge.end) == dist.end()
				|| dist.find(edge.end)->second > edge.weight + dist.find(node.end)->second) { // relax
				dist[edge.end] = edge.weight + dist.find(node.end)->second;
				node_distance.push(edge_adjacent(edge.end, dist[edge.end]));
			}
		}
	}
	return dist;
}
int main(){
#pragma region shortest_path
	graph_adjacent_map graph_map;
	graph_map.nodes[0] = { {1,1} ,{2,2},{3,3} };
	graph_map.nodes[1] = { {2,1} ,{3,4},{4,4} };
	graph_map.nodes[2] = { {3,1} ,{4,2},{4,13} };
	graph_map.nodes[3] = { {4,1} ,{2,2},{1,3} };
	graph_map.nodes[4] = { {0,1} ,{2,2},{3,3} };
	try
	{
		auto result = dijkstra(graph_map, 4);
		for (auto& t : result) {
			std::cout << "distance to " << t.first << " = " << t.second << std::endl;
		}
	}
	catch (const std::exception& e)
	{
		std::cout << e.what() << std::endl;
	}

#pragma endregion

		return 0;
	}
```
- A\*算法：一种针对单源点，单终点的启发式算法，是对Dijkstra算法的一个扩展。其结果并不总是准确的，但可以通过调整达到满意的要求。
	- 基本思路：
		- 待优化公式：对于从源点$s$经过$u$到终点$v$的最短路径估计函数$f[v]$，令$f[v]=d(s,u)+h(u,v)$，其中
			- $d(s,u)$是起点到当前点$u$的最短路径估计
			- $h(u,v)$是当前点$u$到终点$v$的代价估计，是一个启发式函数
		- 每一轮广搜，都选择当前具有最小$d[u]$的节点$v$进行扩展
	- 特例：
		- 当$h(u,v) = 0$，则该方法就是Dijkstra算法
		- 否则，根据应用场景不同，可以选择对应的简单启发函数
			- 图中仅允许四向移动：取曼哈顿距离（水平和垂直距离的和）
			- 图中允许八向移动：使用对角线距离（水平和垂直距离的最大值）
			- 图中允许全向移动：使用欧式距离
		- 其他情况，根据$d$和$h$的大小不同，算法有不同的性能和准确性表现
- Bellman-Ford算法：存在负权边的情况下，解决单源最短路径问题。时间复杂度$O(VE)$。
	- 基本思路：
		1. 初始化最短路径估计
		2. 对每一条边进行一个轮次松弛操作
		3. 重复步骤2共$|V|-1$次，即共$(|V|-1)*|E|$次松弛
		4. 判断任意边$(u,v)$，是否出现$d[v]>d[u]+w(u,v)$
		5. 是：则存在负权环，无最短路径
		6. 否：存在最短路径，返回
	- 正确性证明思路：对于一条路径$p=<v_0,v_1,...,v_k>$，第i轮次的松弛，松弛的边为$(v_{i-1},v_{i})$。根据路径松弛性质可知第i轮能够获得到$v_i$的最短路径。
```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
typedef unsigned int node_id;
typedef int edge_weight;
struct edge_adjacent {
	node_id end;
	edge_weight weight;

	edge_adjacent(node_id end, edge_weight weight) {
		this->end = end;
		this->weight = weight;
	}

	bool operator<(const edge_adjacent& b) {
		this->end = end;
		this->weight = weight;
	}
};

struct node_adjacent {
	node_id begin;
	std::vector<edge_adjacent> edges;
	node_adjacent(node_id begin, std::vector<edge_adjacent>&& edges) {
		this->begin = begin;
		this->edges = edges;
	}
};

struct graph_adjacent {
	std::vector<node_adjacent> nodes;
};

std::unordered_map<node_id, edge_weight> bellman_ford_shortest_path(const graph_adjacent& graph,node_id source) {
	if (graph.nodes.empty()) {
		throw std::exception("empty graph");
	}
	std::unordered_map<node_id, edge_weight> dist;
	dist[source] = 0;
	for (std::size_t i = 1; i != graph.nodes.size(); ++i) {
		for (auto& node : graph.nodes) {
			if (dist.find(node.begin) == dist.end()) { // the begin of this edge is reachable
				continue;
			}
			for (auto& edge : node.edges) {
				if (dist.find(edge.end) == dist.end()
					|| dist.find(edge.end)->second > edge.weight + dist.find(node.begin)->second) { // relax
					dist[edge.end] = edge.weight + dist.find(node.begin)->second;
				}
			}
		}
	}
	// check if has false 
	for (auto& node : graph.nodes) {
		for (auto& edge : node.edges) {
			if (dist.find(edge.end)->second > edge.weight + dist.find(node.begin)->second) {
				throw std::exception("bad graph");
			}
		}
	}
	return dist;
}
int main(){
#pragma region shortest_path
	using namespace lyc_algorithm;
	graph_adjacent graph;
	graph.nodes.push_back(node_adjacent(0, { {1,1} ,{2,2},{3,3} }));
	graph.nodes.push_back(node_adjacent(1, { {2,1} ,{3,4},{4,4} }));
	graph.nodes.push_back(node_adjacent(2, { {3,1} ,{4,2},{4,13} }));
	graph.nodes.push_back(node_adjacent(3, { {4,1} ,{2,2},{1,3} }));
	graph.nodes.push_back(node_adjacent(4, { {0,1} ,{2,2},{3,3} }));
	try
	{
		auto result = bellman_ford_shortest_path(graph, 4);
		for (auto& t : result) {
			std::cout << "distance to " << t.first << " = " << t.second << std::endl;
		}
	}
	catch (const std::exception& e)
	{
		std::cout << e.what() << std::endl;
	}

#pragma endregion

	return 0;
}
```
### 差分约束与最短路径
- 差分约束系统
	- 线性规划矩阵A的每一行包含一个1和一个-1，A的所有其他元素都为0。因此由$Ax \le b$给出的约束条件是m个差分约束集合（m个不等式），包含n个未知元。例如$m=5,n=4$时的一种约束系统可表示为：
	</br><center>
	$\left(\begin{matrix} 1 & -1 & 0 & 0\\\\
	1 & 0 & 0 & -1 \\\\
	0 & 1 & 0 & -1 \\\\ 
	0 & 0 & 1 & -1 \\\\ 
	-1 & 0 & 1 & 0 \\\\ \end{matrix}\right) 
	\left(\begin{matrix} x_1 \\\\ x_2 \\\\ x_3 \\\\ x_4  \end{matrix}\right)
	\le 
	\left(\begin{matrix} 0 \\\\ 1 \\\\ 2 \\\\ 3 \\\\ 4 \end{matrix}\right)$ 
	</center>

	- 和图算法的关系：可以将差分约束转换为最短路径问题。
		- 转换为图$G=(V,E)$，其中$V=\\{v_0,v_1,...,v_n\\}$,$E=\\{(v_i,v_j):x_j-x_i\le b_k\mathrm{是一个约束条件}\\}\cup \\{(v_0,v_1),(v_0,v_2),...,(v_0,v_n)\\}$。另外对于约束$x_j-x_i\le b_k$，边$(v_i,v_j)$具有权$w(v_i,v_j)=b_k$，而其他边则有$w(v_0,v_i)=0,1\le i \le n$。
		- 引入逻辑顶点$v_0$保证每个顶点均可从$v_0$到达。其他顶点就是未知元，边代表具有约束关系。
	- 给定一差分约束系统$Ax\le b$，设$G=(V,E)$为其相应的约束图。如果$G$不包含负权回路。则系统的一个可行解为：
	</br><center>
	$x=(\delta(v_0,v_1),\delta(v_0,v_2),...,\delta(v_0,v_n))$
	</center>

	- 证明思路：可以直接将权值的定义和可行解带入，易知其确为可行解。
### 多源最短路
- 当我们可以计算单源点最短路之后，很显然，重复$|V|$次就是所有源点可得到的最短路。但这个问题还有一些更好，或者更直观的办法。
- 术语：

- Floyd算法（Floyd-Warshall算法）：时间复杂度$\Theta(V^3)$
	- 基本思路：
		1. 遍历取出节点记为$k$
		2. 对每一个$k$，遍历点对$(i,j)$，尝试用$i\leadsto k,k\leadsto j$，更新$i\leadsto j$的最小路径估计
	- 正确性证明思路：
		- 对$k$，考虑顶点子集$\\{v_1,v_2,...,v_k\\}$，以及所有路径$i\leadsto j$中，中间节点均在此集合的路径，且设$p$是其中的最小权值路径。
		- 如果$k$不是$p$的中间顶点，则$p$的所有顶点在$\\{v_1,v_2,...,v_{k-1}\\}$中。不影响。
		- 如果$k$是$p$的中间顶点，则$p$可分解为$i\leadsto k \leadsto j$，两部分，而根据定义前后两个子路径已经是对应顶点间的最短路径（估计）。
		- 归纳法，可以得知当迭代到$k$取所有值，则可计算出真正的全部最短路径权值。
```cpp
#include<iostream>
#include<vector>
#include<xutility>
int main(){
	std::vector<std::vector<int>> graph={{0,1,4,3},{1,0,4,5},{-1,2,0,3},{2,4,3,0}};
	std::vector<std::vector<int>> dst=graph;
	for(int k=0;k!=graph.size();++k){
		for(int i=0;i!=graph.size();++i){
			if(i==k)
				continue;
			for(int j=0;j!=graph.size();++j){
				if(i==j||k==j)
					continue;
				dst[i][j]=std::min(dst[i][j],dst[i][k]+dst[k][j]);
			}
		}
	}
	for (int i = 0; i != dst.size(); ++i) {
		for (int j = 0; j != dst.size(); ++j) {
			std::cout << dst[i][j] << " ";
		}
		std::cout << std::endl;
	}
	return 0;
}
```
- Johnson算法：简单版时间复杂度可达$O(VElogV)$，[斐波那契堆]({{<relref "/content/post/CSBasic/algo7-tree.md#斐波那契堆">}})优化可达$O(V^2logV+VE)$，稀疏图更快
	- 重赋权技术：
		- 如果图$G=(V,E)$中所有边的权$w$都是非负的，则执行$|V|$次Dijkstra算法即可。
		- 如果存在负权，则重新设定一个权值函数$\hat{w}$，满足：路径$p$是加权函数$w$从$u$到$v$的一条最短路径，当且仅当$p$也是加权函数$\hat{w}$从$u$到$v$的一条最短路径；加权函数$\hat{w}$的值永远非负。
		- 构造方式：
			1. 构造新图$G'=(V',E')$，其中$V'=V\cup\\{s\\}$，$E'=E\cup\\{(s,v):v\in V\\}$。即添加一个超级源点$s$，连接所有其他顶点。且扩展原有加权函数$w$，使得$w(s,v)=0,v\in V$。
			2. 设函数$h(v)=\delta(s,v)$，即先计算出超级源点到其他点的最短路，由三角不等式易知$h(v)\le h(u)+w(u,v)$，定义新权$\hat{w}(u,v)=w(u,v)+h(u)-h(v)$。注意$u\in V'$。
		- 构造正确性的证明思路：
			- 新权$\hat{w}(u,v)$的非负性很显然（带入$h$的定义,结合三角不等式）
			- $w,\hat{w}$指示同一条最短路径：带入定义，展开$\hat{w}(p)=\sum_{i=1}^{k}\hat{w}(v_{i-1},v_i)$，可知$\hat{w}(p)=w(p)+h(v_0)-h(v_k)$。$h$是固定值，易知两者取得最短路径确实是当且仅当的。
	- 为什么要重赋权？在稀疏图的语境下，Djikstra比Bellman-ford更快，但是不能应用于带有负权边的图。
	- 基本思路：
		1. 调用bellman-ford算法计算构造图$G'=(V',E')$中，以$s$为源点的最短路径权值
		2. 根据步骤1中结果，设定$h$，并构造$\hat{w}$
		3. 基于$\hat{w}$，对每一个顶点调用djikstra算法，得到结果$\hat{\delta}(u,v)$
		4. 还原$\delta(u,v)$
```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>
#include<unordered_set>
#include"dijkstra.hpp"
#include"bellman_ford.hpp"

namespace lyc_algorithm {
	struct edge_adjacent {
		node_id end;
		edge_weight weight;

		edge_adjacent(node_id end, edge_weight weight) {
			this->end = end;
			this->weight = weight;
		}

		bool operator<(const edge_adjacent& b) const {
			return this->weight < b.weight;
		}
	};

	typedef edge_adjacent distance_single_source;

	struct node_adjacent {
		node_id begin;
		std::vector<edge_adjacent> edges;
		node_adjacent(node_id begin, std::vector<edge_adjacent>&& edges) {
			this->begin = begin;
			this->edges = edges;
		}
	};

	struct graph_adjacent {
		std::vector<node_adjacent> nodes;
	};

	struct graph_adjacent_map {
		std::unordered_map<node_id, std::vector<edge_adjacent>> nodes;
	};

	template<typename T>
	struct min_heap {
		bool operator()(const T& a, const T& b) {
			if (!(a < b) && !(b < a)) // strict weak order
				return false;
			else
				return !(a < b);
		}
	};

	std::unordered_map<node_id, edge_weight> dijkstra(const graph_adjacent_map& graph,node_id source) {
		std::unordered_map<node_id, edge_weight> dist;
		std::priority_queue<edge_adjacent, std::vector<edge_adjacent>, min_heap<edge_adjacent>> node_distance;
		std::unordered_set<node_id> arrived_nodes;
		node_distance.push(edge_adjacent(source, 0));
		dist[source] = 0;
		while (!node_distance.empty()) {
			auto node = node_distance.top();
			node_distance.pop();
			if (arrived_nodes.find(node.end) != arrived_nodes.end()) {
				continue;
			}
			// new node
			arrived_nodes.insert(node.end);
			for (auto edge : graph.nodes.find(node.end)->second) {
				if (dist.find(edge.end) == dist.end()
					|| dist.find(edge.end)->second > edge.weight + dist.find(node.end)->second) { // relax
					dist[edge.end] = edge.weight + dist.find(node.end)->second;
					node_distance.push(edge_adjacent(edge.end, dist[edge.end]));
				}
			}
		}
		return dist;
	}

	std::unordered_map<node_id, edge_weight> bellman_ford_shortest_path(const graph_adjacent_map& graph, node_id source) {
		if (graph.nodes.empty()) {
			throw std::exception("empty graph");
		}
		std::unordered_map<node_id, edge_weight> dist;
		dist[source] = 0;
		for (std::size_t i = 1; i != graph.nodes.size(); ++i) {
			for (auto& node : graph.nodes) {
				if (dist.find(node.first) == dist.end()) { // the begin of this edge is reachable
					continue;
				}
				for (auto& edge : node.second) {
					if (dist.find(edge.end) == dist.end()
						|| dist.find(edge.end)->second > edge.weight + dist.find(node.first)->second) { // relax
						dist[edge.end] = edge.weight + dist.find(node.first)->second;
					}
				}
			}
		}
		// check if has false 
		for (auto& node : graph.nodes) {
			for (auto& edge : node.second) {
				if (dist.find(edge.end)->second > edge.weight + dist.find(node.first)->second) {
					throw std::exception("bad graph");
				}
			}
		}
		return dist;
	}

	std::unordered_map<node_id,std::unordered_map<node_id,edge_weight>> johnson(const graph_adjacent_map& graph) {
		// create G'
		graph_adjacent_map graph_ = graph;
		node_id id;
		for (id = 0; id <= graph_.nodes.size(); ++id) {
			if (graph_.nodes.find(id) == graph_.nodes.end()) {
				for (auto& t : graph.nodes) {
					graph_.nodes[id].push_back(edge_adjacent(t.first, 0));
				}
				break;
			}
		}
		auto h = bellman_ford_shortest_path(graph_, id);
		// config w' base on h & w
		for (auto& node : graph_.nodes) {
			for (auto& edge : node.second) {
				edge.weight = edge.weight + h[node.first] - h[edge.end];
			}
		}
		std::unordered_map < node_id, std::unordered_map<node_id, edge_weight>> dist;
		for (auto& node : graph_.nodes) {
			if (node.first == id)
				continue;
			auto result = dijkstra(graph_, node.first);
			for (auto& dist : result) {
				dist.second = dist.second - h[node.first] + h[dist.first];
			}
			dist[node.first] = result;
		}
		return dist;
	}
}
int main(){
	using namespace lyc_algorithm;
	graph_adjacent_map graph_map;
	graph_map.nodes[0] = { {1,1} ,{2,2},{3,3} };
	graph_map.nodes[1] = { {2,1} ,{3,4},{4,4} };
	graph_map.nodes[2] = { {3,1} ,{4,2},{0,-2} };
	graph_map.nodes[3] = { {4,1} ,{2,2},{1,3} };
	graph_map.nodes[4] = { {0,1} ,{2,2},{3,3} };
	auto result = johnson(graph_map);
	for (auto& node : result) {
		std::cout << "from node: " << node.first << std::endl;
		for (auto& end : node.second) {
			std::cout << "    to node: " << end.first << " -> " << end.second << std::endl;
		}
	}
	return 0;
}
```
## 最大流问题
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

## 求图的割边集
割边集，即图中所有的桥。删除这些边，将会增大图的联通子图数量。内容在[算法进阶]({{<relref "/content/post/CSBasic/algo10-advance.md#图论-求桥">}})给出。

## 回收封面
- 首先很感谢你有耐心看到这里。这一小节的内容是想把封面回收。计算机领域的一个经典问题就是P与NP问题。
- 术语：
	- P问题（Polynomial）：具有多项式时间求解算法的问题。即复杂度形如$O(n^a)$，$a$为常数。
	- NP问题（Nondeterminism Polynomial）：具有能在多项式时间内验证结果正确的算法的问题。注意NP问题的描述范围，只用于验证判定类问题（比如一个路径的长度是不是t），而不是直接适用于描述验证一个解是否是最优解等优化问题。
	- NPC问题（NP Complete，完全NP问题）：存在这样一个NP问题，所有的NP问题都可以规约为它（即只要解决了这个问题，其他NP问题的解也可以在多项式时间内转换而同时得出）。它就是NPC问题。
	- NPH问题（NP Hard，NP难问题）：所有NP问题都可以归约到该问题，但不一定是一个NP问题，即该问题复杂度可能更高，多项式时间内无法验证。
- 遗憾的是，我们还是不能知道P=NP成立与否。这两个集合之间的关系还等待确认。但很有趣的事情是，很多NP问题之间，看起来非常相似。
- 旅行商问题：在一个带权重无负环的强连通图中，访问每个顶点恰好一次，并回到初始点，求这种访问方式中，权值和最小的走法。
	- 至少目前而言，旅行商问题是NP完全的（这里的验证，是给定了路径和权值和之后进行验证）
- 最长简单路径问题：虽然我们知道最短路径问题是一个P问题，但是很有趣的是，最长简单路径问题是NP完全的。甚至即使所有边权值是1，它也是NP完全的。（最最长的，是把所有点走一遍）