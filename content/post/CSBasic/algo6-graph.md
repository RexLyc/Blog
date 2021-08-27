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
2. 度：
    - 入度：
    - 出度：
3. $G=(V,E)$：表示一个以V为顶点集，E为边集的图。
4. 前驱：路径$(u,v)$中，如果顶点$i$在$j$前面访问，则称$i$是$j$的前驱。
5. 后继：和前驱相反，如果顶点$i$在$j$前面访问，则称$j$是$i$的后继。
# 数据结构
1. 邻接矩阵：对于一个无权图$G=(V,E)$（有权图同理），维护一个矩阵$A=(a_{ij})$，其中
</br><center>$a_{ij} = \left\\{ \begin{array}{11}
    1 & \mathrm{如果}(i,j)\in E\\\\
    0 & \mathrm{否则}
 \end{array} \right.$</center>
 
2. 邻接表：对于一个无权图$G=(V,E)$（有权图同理），维护一个包含$|V|$个列表的数组$A$组成，该数组的每一个元素$A[u]$是一个列表，列表中的元素$v$，代表$(u,v)\in E$。
> 邻接矩阵并不一定就浪费空间，如果是无权图，可以用位运算的形式压缩，每个标记只占一个bit，只是这样做确实很麻烦。</br>另外只要是无向图，就可以只用一半的矩阵空间，对于java这种天生支持异形数组的语言来说还是ok的。就是用的时候带着一股邪气。
# 图搜索
1. 广度优先搜索（BFS：breadth-first search）
- 基本思路：算法每一层的搜索，都会沿着已发现的节点的边界，访问所有到当前边界距离为1的所有未发现顶点。
- 实现思路：广度优先搜索天生适合用队列进行实现，其遍历过程等价于队列节点的入队出队过程。
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
- 基本思路：扩展当前已访问集合时，以特定规则从某点开始，沿着一条路径一直扩展顶点，直到无法继续。
- 实现思路：深度优先搜索天生适合用栈实现（不喜欢实现栈直接递归调用即可），扩展节点的过程是入栈，递归返回的过程是出栈。
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
    //表格中1为可走的地点，0为不可走的（路径和墙）。
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
- 对于有向图而言，图的拓扑是一个非常有用的基础信息。
# 图分解
# 生成树问题
# 最短路径问题
# 最大流问题