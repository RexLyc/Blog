---
title: "算法导论其七：树"
date: 2021-08-27T10:30:52+08:00
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
树是一类特殊的图，但和图不一样的是，树作为数据结构的应用十分广泛。还有各种各样的变种用于加速一些特定领域的算法。性能是相当的妙啦~
<!--more-->
# 有根树
- 实际上只是一种数据结构。用链接的方式，构造出一个树状的数据结构。
- 术语：
    - 根：有根树的最上层节点，在树中，只有这个节点是没有双亲结点的
    - 子树：树的定义是递归的，每一个节点和它的子孙节点都构成整个树的一棵子树
    - 度：每个节点的**子节点的个数**称为该节点的度，树的度取所有节点的度中最大的
    - 深度：从根节点向下逐层累加。
    - 高度：从最底层叶子节点自底向上累加。
        - 对于每个节点来说，深度和高度不一定相等，但一棵树的深度和高度相等。
    - 域：$p,left,right$，分别用于指示二叉树中的双亲、左孩子、右孩子，例如：
        - $p\[x\]= NIL$，则代表x是根。$p\[x\]\gets y$，则表示将y设置为x的双亲节点（parent缩写为p）
        - $left\[x\]= NIL$，代表x没有左孩子。$left\[x\]\gets y$，表示将y设置为x的左孩子
    - 扩展的域：$right-sibling$，指示二叉树中最近邻的右侧的兄弟（位于树的同一层的都是兄弟节点）
    - 一些常用的树：
        - 二叉树：每个节点的度$\le 2$。
        - 二叉查找（二叉搜索树）：对任何节点$t$，若左子树存在，则左子树节点值永远不大于$t$的值。若右子树存在，则右子树的值永远不小于$t$的值。
        - 平衡搜索树（AVL树）：二叉搜索树的一种，并且满足任何左子树和右子树高度之差不会大于1。
        - 满二叉树：除了叶子节点，所有节点度为2。
        - 完全二叉树：最后一层叶子节点均在左侧的满二叉树。
        - 完美二叉树：每一层均被填满的二叉树。
```cpp
// 一个典型的指针实现的
template<typename T>
struct tree_node {
    tree_node *left, *right;
    tree_node *parent;
    T val;
    
    tree_node(T val)
        : left(nullptr), right(nullptr), parent(nullptr),val(val) {}

    tree_node(T val, tree_node *left, tree_node *right, tree_node *parent)
        : val(val), left(left), right(right), parent(parent) {}
};
```
# 二叉查找树（二叉搜索树）
- 普通的有根树有什么用吗？没有。节点之间只有满足一些性质的时候，树才会显现出威力。
- 二叉查找树是查找树的一种，定义已在上一小节给出。
- 提供的核心操作：插入、删除、查询。一个良好实现的二叉查找树均能实现$O(logn)$的复杂度，$n$为树的高度。
- 其他基础操作：子树最小元素（Tree-Minimum）、子树最大元素（Tree-Maximum）、节点中序遍历下的前驱（Tree-Predecessor）、节点中序遍历下的后继（Tree-Successor）
- 常见的实用二叉搜索树：
    1. AVL树：得名于其发明者的名字（ Adelson-Velskii 以及 Landis）。AVL树是最先发明的严格自平衡二叉查找树，其任何节点的左右子树高度差绝对值小于1。
    2. 红黑树：下面就讲啦。
# 红黑树
- 一种非常使用的二叉查找树的实现。保证最坏的情况下依然满足插入、删除、查询的操作时间为$O(logn)$。
- 红黑树保证没有一条路径的长度会比其他路径长出两倍，因而是接近平衡的。红黑树这种宽松的平衡条件，比AVL树的绝对平衡，在插入和删除时有性能优势，但在查询时是处于劣势的。
- 充要条件：
    1. 每个节点或者是红色，或者是黑色
    2. 根节点是黑色的
    3. 每个叶子节点（NIL）是黑色的，叶子节点不存储数据，存储节点的数据也被称为内部节点
    4. 如果一个节点是红色的，则它的孩子节点都是黑色的
    5. 对于每一个节点，从该节点到其子孙叶子节点的所有路径上包含相同数目的黑节点
- 核心操作：旋转和重新着色。目标是在树发生结构变化时，令树继续保持充要条件。
    1. 旋转：旋转操作实现时，只需要进行指针变换，左右旋转都可以在$O(1)$内完成。
    </br><center>
    ![树的旋转操作](/images/algoSeries/TreeRotate.svg)
    </center>
    
    2. 重新着色：每一次新节点插入时，以基础的二叉查找树的插入方式，将新节点插入书中，并固定的设置新节点的颜色为红色。这会导致新树不满足原有的红黑树在着色方面的要求。同理，删除节点时也会破坏原有性质。此时将会根据情况重新对部分节点进行重新上色（红色变黑色，黑色变红色）。
- 插入：在二叉查找树的插入操作的基础上，添加重新着色和旋转操作。向树$T$插入$z$的步骤如下：
    1. 自根向下查找大小适合插入$z$的位置，并插入
        - 空树则$z$直接为根
    2. 设置$z$为红色
    3. 插入的重着色和旋转：
        1. 循环2或3直到$color[p[z]]=BLACK$
        2. 父节点为左孩子
            1. 叔父节点红色：情形一，$z$上移
            2. 叔父节点黑色，$z$为右孩子：情形二，变更$z$
            3. 情形三
        3. 父节点为右孩子
            - 处理方式为镜像情况
        4. 将根赋值为黑色

    </br><center>
    ![红黑树的插入操作](/images/algoSeries/RB-Tree-INSERT.svg)
    </br>三种插入情形
    </center>
    
- 删除：删除操作也是在二叉查找树的删除操作的基础上，进行修改得来的。重新着色和旋转情况与插入相比，情况稍微复杂。从树$T$删除$z$的步骤如下：
    1. 寻找待删除节点$y$
        - 若$left[z]=nil[T]$或$right[z]=nil[T]$，则$y\gets z$
        - 否则$y\gets Tree-Successor(z)$，即实际删除中序遍历下的后继节点（后续会将当前$z$节点的值替换为后继）
    2. 计算保留节点$x$，$x$将会代替$y$的位置，并令$p\[x\]\gets p[y]$
        - 若$left[y]\ne nil[T]$，则$x\gets left[y]$
        - 否则$x\gets right[y]$（$x$可能为$nil[T]$）
    3. 调整$p[y]$的$left$，$right$域
        - 若$y$为根，令$x$为根
        - 若$y$不为根，且为左孩子，则$x$也为左孩子，反之亦然。
    4. 若$y\ne z$（意味着取了后继），$key[z]\gets key[y]$，即将卫星数据拷贝过去。
    5. 若$color[y]=BLACK$，进行删除的重着色和旋转（对保留下来的$x$进行）
        1. 若$x\ne root[T]$且$color\[x\]=BLACK$，循环处理2或3
        2. 若$x$为左孩子
            1. 若$x$的兄弟$w$为红色，情形一
            2. 若$x$的兄弟$w$为黑色，且$w$的左右孩子均为黑色，情形2
            3. 若$x$的兄弟$w$为黑色，且$w$的右孩子为黑色，情形3
            4. 情形4（C取A原色）
        3. 若$x$为右孩子，操作均为镜像
        4. $color\[x\]\gets BLACK$
    
    </br><center>
    ![红黑树的删除操作](/images/algoSeries/RB-Tree-Delete.svg)
    </br>四种删除情形(绿色代表该节点颜色无所谓)
    </br>注：情形4中C取A色，D色不变
    </center>

- 关于插入和删除の一点理解：重新着色和旋转操作都是为了恢复红黑树的性质，对于每种情况，要看懂到能够理解违反了哪项性质，操作之后是否恢复，或者是否让子树部分恢复，并递归向上。
    - 插入的问题是两个红色相邻
    - 删除的主要问题是，**包含删除节点的任意路径，黑高度降低**。（另外还可能有其他性质被打破）
        - 这里主要需要思考清楚的是情形一和二，这两者不会重复循环出现：若是从情形一到情形二，则将会很快退出循环；若是从情形二到情形一，则至多为二到一再到二，并退出
        - 一旦进入情形三、四，将会很快退出
```cpp
// 这玩意儿能写好久
#include<vector>
#include<iostream>
namespace lyc_algorithm {

	template<typename T>
	struct binary_tree_node {
		binary_tree_node* parent;
		binary_tree_node* left;
		binary_tree_node* right;
		T data;
		// 数据存储次数
		std::size_t times;
		binary_tree_node(binary_tree_node* parent
			, binary_tree_node* left, binary_tree_node* right, const T& data)
			:parent(parent), left(left), right(right), data(data),times(1) {}
		binary_tree_node()
			:parent(nullptr), left(nullptr), right(nullptr),times(0) {}
		binary_tree_node(binary_tree_node* parent)
			:parent(parent), left(nullptr), right(nullptr), times(0) {}
	};



	enum class NODE_COLOR {
		RED, BLACK
	};

	std::string enum_trans(const NODE_COLOR& e) {
		switch (e) {
			case NODE_COLOR::RED:
				return "RED";
			case NODE_COLOR::BLACK:
				return "BLACK";
			default:
				return "??";
		}
	}

	template<typename T>
	struct binary_search_tree_node : public binary_tree_node<T> {
		binary_search_tree_node(binary_search_tree_node* parent
			, binary_search_tree_node* left, binary_search_tree_node* right, const T& data)
			:binary_tree_node<T>(parent, left, right, data) {}
		binary_search_tree_node()
			:binary_tree_node<T>() {}
		binary_search_tree_node(binary_search_tree_node* parent)
			:binary_tree_node<T>(parent) {}
	};

	template<typename T>
	struct redblack_tree_node: public binary_search_tree_node<T> {
		// 创建一个内部节点
		redblack_tree_node(redblack_tree_node* parent
			, const T& data,  NODE_COLOR color = NODE_COLOR::RED)
			:binary_search_tree_node<T>(parent
				, new redblack_tree_node(this), new redblack_tree_node(this), data)
			, color(color) ,is_nil(false){}

		// 创建一个无数据的叶子节点
		redblack_tree_node(redblack_tree_node* parent)
			:binary_search_tree_node<T>(parent), color(NODE_COLOR::BLACK) ,is_nil(true) {}

		NODE_COLOR color;
		bool is_nil;

		// 废弃：待删除节点应当直接被删除，而不是转为叶子节点
		void to_nil() {
			if (is_nil) {
				throw std::exception("this node is already a nil");
			}
			else if (!(to_rbnode(binary_tree_node<T>::left)->is_nil)
				|| !(to_rbnode(binary_tree_node<T>::right)->is_nil)) {
				throw std::exception("to_nil should only apply to the node which has two nil childs");
			}
			else {
				delete binary_tree_node<T>::left;
				delete binary_tree_node<T>::right;
				binary_tree_node<T>::times = 0;
				is_nil = true;
				color = NODE_COLOR::BLACK;
			}
		}

		// 将叶子节点转为内部节点，并自动创建新的叶子节点
		void to_inner(const T& data) {
			if (is_nil) {
				binary_tree_node<T>::data = data;
				binary_tree_node<T>::times = 1;
				is_nil = false;
				this->color = NODE_COLOR::RED;
				binary_tree_node<T>::left = new redblack_tree_node<T>(this);
				binary_tree_node<T>::right = new redblack_tree_node<T>(this);
			}
			else {
				throw std::exception("this node is already a inner node");
			}
		}

		// 析构时删除存在的叶子孩子节点
		~redblack_tree_node() {
			if (binary_tree_node<T>::left && to_rbnode(binary_tree_node<T>::left)->is_nil) {
				delete binary_tree_node<T>::left;
			}
			if (binary_tree_node<T>::right && to_rbnode(binary_tree_node<T>::right)->is_nil) {
				delete binary_tree_node<T>::right;
			}
		}
	};

	template<typename T>
	constexpr redblack_tree_node<T>* to_rbnode(binary_tree_node<T>* node) {
		return reinterpret_cast<redblack_tree_node<T>*>(node);
	}

	// 左旋：返回左旋后新的根
	template<typename T>
	binary_tree_node<T>* left_rotate(binary_tree_node<T>* parent) {
		binary_tree_node<T>* right = parent->right;
		binary_tree_node<T>* parent_parent = parent->parent;
		if (!right) // 不存在右孩子，结构不支持左旋
			return nullptr;
		// 新根和老根相关孩子节点关系变更
		parent->right = right->left;
		if (right->left) {
			right->left->parent = parent;
		}
		parent->parent = right;
		right->left = parent;
		right->parent = parent_parent;
		// 建立新根到原根的父节点之间的关系
		if (parent_parent) {
			if (parent == parent_parent->left) {
				parent_parent->left = right;
			}
			else {
				parent_parent->right = right;
			}
		}
		return right;
	}

	// 右旋，返回右旋后新的根
	template<typename T>
	binary_tree_node<T>* right_rotate(binary_tree_node<T>* parent) {
		binary_tree_node<T>* left = parent->left;
		binary_tree_node<T>* parent_parent = parent->parent;
		if (!left)
			return nullptr;
		parent->left = left->right;
		if (left->right) {
			left->right->parent = parent;
		}
		parent->parent = left;
		left->right = parent;
		left->parent = parent_parent;
		if (parent_parent) {
			if (parent == parent_parent->left) {
				parent_parent->left = left;
			}
			else {
				parent_parent->right = left;
			}
		}
		return left;
	}

	// 在二叉搜索树中查询指定数据，返回nullptr，或空数据节点，或包含当前data的节点
	template<typename T>
	binary_search_tree_node<T>* query(binary_search_tree_node<T>* root, const T& data) {
		binary_search_tree_node<T>* current = root;
		while (current&&current->times&&current->data!=data) {
			if (current->data > data) {
				current = reinterpret_cast<binary_search_tree_node<T>*>(current->left);
			}
			else {
				current = reinterpret_cast<binary_search_tree_node<T>*>(current->right);
			}
		}
		return current;
	}

	// 返回中序遍历下的节点指针列表
	template<typename T>
	void inorder(binary_tree_node<T>* root, std::vector<binary_tree_node<T>*>& in_order_list) {
		if (!root || !root->times)
			return;
		inorder(root->left, in_order_list);
		in_order_list.push_back(root);
		inorder(root->right, in_order_list);
	}

	// 获取当前根所带子树下的最小值节点，不存在（空树）返回nullptr，存在则返回对应节点
	template<typename T>
	binary_search_tree_node<T>* tree_minimum(binary_search_tree_node<T>* root) {
		binary_tree_node<T>* current = root;
		binary_tree_node<T>* memory = nullptr;
		while (current && current->times) {
			memory = current;
			current = current->left;
		}
		return reinterpret_cast<binary_search_tree_node<T>*>(memory);
	}

	/*
	* return the last data node that is after data in the inorder
	*/
	// 返回中序遍历下data的后继，要求data必须存在于树中
	template<typename T>
	binary_search_tree_node<T>* successor(binary_search_tree_node<T>* root, const T& data) {
		binary_tree_node<T>* current = query(root,data);
		if (!current || !current->times)
			throw std::exception("data not in tree, can't search successor");
		// 有右子树，返回右子树中最小值
		if (current->right && current->right->times)
			return tree_minimum(reinterpret_cast<binary_search_tree_node<T>*>(current->right));
		// 否则需要找到父结点中第一个比自己大的
		binary_tree_node<T>* memory = current->parent;
		while (memory && memory->right==current) {
			current = memory;
			memory = memory->parent;
		}
		return reinterpret_cast<binary_search_tree_node<T>*>(memory);
	}

	// 插入后恢复红黑树
	template<typename T>
	void redblack_tree_insert_fix(redblack_tree_node<T>** root, redblack_tree_node<T>* new_node) {
		redblack_tree_node<T>* current_node = new_node;
		// current始终指向一个红节点，因此退出条件为：抵达根节点，或者父节点不红
		while (current_node->parent && to_rbnode(current_node->parent)->color == NODE_COLOR::RED) {
			if (current_node->parent == current_node->parent->parent->left) { // 父节点为左孩子
				if (current_node->parent->parent->right 
					&& to_rbnode(current_node->parent->parent->right)->color == NODE_COLOR::RED) {
					// 情形一
					to_rbnode(current_node->parent->parent->right)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->parent)->color = NODE_COLOR::RED;
					current_node = to_rbnode(current_node->parent->parent);
				}
				else if (current_node == current_node->parent->right) {
					// 情形二
					current_node = to_rbnode(left_rotate(current_node->parent)->left);
				}
				else {
					// 情形三
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->parent)->color = NODE_COLOR::RED;
					// 若对根旋转，需要更新根结点的值
					if (current_node->parent->parent == *root)
						*root = to_rbnode(right_rotate(current_node->parent->parent));
					else
						right_rotate(current_node->parent->parent);
				};
			}
			else { // 镜像情况，将所有左、左旋，替换为右、右旋，反之亦然
				if (current_node->parent->parent->left 
					&& to_rbnode(current_node->parent->parent->left)->color == NODE_COLOR::RED) {
					to_rbnode(current_node->parent->parent->left)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->parent)->color = NODE_COLOR::RED;
					current_node = to_rbnode(current_node->parent->parent);
				}
				else if (current_node == current_node->parent->left) {
					current_node = to_rbnode(right_rotate(current_node->parent)->right);
				}
				else {
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->parent)->color = NODE_COLOR::RED;
					if (current_node->parent->parent == *root)
						*root = to_rbnode(left_rotate(current_node->parent->parent));
					else
						left_rotate(current_node->parent->parent);
				}
			}
		}
		// 针对回到根的情况，需要保证根是黑色
		(*root)->color = NODE_COLOR::BLACK;
	}

	// 向红黑树插入数据
	template<typename T>
	void redblack_tree_insert(redblack_tree_node<T>** root, const T& data) {
		// 空树直接插入
		if (!*root) {
			*root = new redblack_tree_node<T>(nullptr, data, NODE_COLOR::BLACK);
			return;
		}
		// 查询是否已经存在
		redblack_tree_node<T>* query_node = to_rbnode(query(*root, data));
		if (!query_node->is_nil) {
			query_node->times++;
			return;
		}
		// 不存在的情况下，query返回的是恰好可用于插入的空数据节点，提升为内部节点
		redblack_tree_node<T>* insert_node = query_node;
		insert_node->to_inner(data);
		// 恢复红黑树性质
		redblack_tree_insert_fix(root, insert_node);
	}

	// 删除红黑树节点后的性质恢复
	template<typename T>
	void redblack_tree_delete_fix(redblack_tree_node<T>** root, redblack_tree_node<T>* current_node) {
		// 退出条件为抵达根节点，或当前节点不为黑色
		while (current_node != *root && current_node->color == NODE_COLOR::BLACK) {
			if (current_node == current_node->parent->left) { // 当前节点是左孩子
				if (to_rbnode(current_node->parent->right)->color == NODE_COLOR::RED
					&& to_rbnode(current_node->parent->right->left)->color == NODE_COLOR::BLACK
					&& to_rbnode(current_node->parent->right->right)->color == NODE_COLOR::BLACK) {
					// 情形一
					to_rbnode(current_node->parent)->color = NODE_COLOR::RED;
					to_rbnode(current_node->parent->right)->color = NODE_COLOR::BLACK;
					if (current_node->parent == *root)
						*root = to_rbnode(left_rotate(current_node->parent));
					else
						left_rotate(current_node->parent);
				}
				else if (to_rbnode(current_node->parent->right)->is_nil
					|| (to_rbnode(current_node->parent->right)->color == NODE_COLOR::BLACK
						&& to_rbnode(current_node->parent->right->left)->color == NODE_COLOR::BLACK
						&& to_rbnode(current_node->parent->right->right)->color == NODE_COLOR::BLACK)) {
					// 情形二
					to_rbnode(current_node->parent->right)->color = NODE_COLOR::RED;
					current_node = to_rbnode(current_node->parent);
				}
				else if (to_rbnode(current_node->parent->right)->color == NODE_COLOR::BLACK
					&& to_rbnode(current_node->parent->right->right)->color == NODE_COLOR::BLACK) {
					// 情形三
					to_rbnode(current_node->parent->right->left)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->right)->color = NODE_COLOR::RED;
					if (current_node->parent->right == *root)
						*root = to_rbnode(right_rotate(current_node->parent->right));
					else
						right_rotate(current_node->parent->right);
				}
				else {
					// 情形四
					to_rbnode(current_node->parent->right)->color 
						= to_rbnode(current_node->parent)->color;
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->right->right)->color = NODE_COLOR::BLACK;
					if (current_node->parent == *root)
						*root = to_rbnode(left_rotate(current_node->parent));
					else
						left_rotate(current_node->parent);
				}
			}
			else {
				if (to_rbnode(current_node->parent->left)->color == NODE_COLOR::RED
					&& to_rbnode(current_node->parent->left->right)->color == NODE_COLOR::BLACK
					&& to_rbnode(current_node->parent->left->left)->color == NODE_COLOR::BLACK) {
					to_rbnode(current_node->parent)->color = NODE_COLOR::RED;
					to_rbnode(current_node->parent->left)->color = NODE_COLOR::BLACK;
					if (current_node->parent == *root)
						*root = to_rbnode(right_rotate(current_node->parent));
					else
						right_rotate(current_node->parent);
				}
				else if (to_rbnode(current_node->parent->left)->is_nil
					|| (to_rbnode(current_node->parent->left)->color == NODE_COLOR::BLACK
						&& to_rbnode(current_node->parent->left->right)->color == NODE_COLOR::BLACK
						&& to_rbnode(current_node->parent->left->left)->color == NODE_COLOR::BLACK)) {
					to_rbnode(current_node->parent->left)->color = NODE_COLOR::RED;
					current_node = to_rbnode(current_node->parent);
				}
				else if (to_rbnode(current_node->parent->left)->color == NODE_COLOR::BLACK
					&& to_rbnode(current_node->parent->left->left)->color == NODE_COLOR::BLACK) {
					to_rbnode(current_node->parent->left->right)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->left)->color = NODE_COLOR::RED;
					if (current_node->parent->left == *root)
						*root = to_rbnode(left_rotate(current_node->parent->left));
					else
						left_rotate(current_node->parent->left);
				}
				else {
					to_rbnode(current_node->parent->left)->color 
						= to_rbnode(current_node->parent)->color;
					to_rbnode(current_node->parent)->color = NODE_COLOR::BLACK;
					to_rbnode(current_node->parent->left->left)->color = NODE_COLOR::BLACK;
					if (current_node->parent == *root)
						*root = to_rbnode(right_rotate(current_node->parent));
					else
						right_rotate(current_node->parent);
					current_node = *root;
				}
			}
		}
		// 保持根为黑，或者对当前红节点添加黑色（完成对缺失的黑色的补充）
		current_node->color = NODE_COLOR::BLACK;
	}

	// 从红黑树中删除指定数据
	template<typename T>
	bool redblack_tree_delete(redblack_tree_node<T>** root, const T& data) {
		redblack_tree_node<T>* node=to_rbnode(query(*root, data));
		// 数据不存在
		if (node->is_nil)
			return false;
		// 数据有多份，暂时不用删除节点
		if (node->times > 1) {
			node->times--;
			return true;
		}
		// 寻找删除节点
		redblack_tree_node<T>* remove_node=nullptr;
		if (to_rbnode(node->left)->is_nil 
			|| to_rbnode(node->right)->is_nil) {
			remove_node = node;
		}
		else {
			// 如果节点有双孩子，则不会被直接删除，而会被其后继节点数据替代，后继节点会被删除
			// 后继节点可以被删除的原因是，在当前节点双孩子的前提下，后继节点至多有一个右孩子
			remove_node = to_rbnode(successor(*root, node->data));
		}
		// 寻找保留节点（后继节点的左孩子或右孩子）
		redblack_tree_node<T>* save_node = nullptr;
		if (to_rbnode(remove_node->left)->is_nil) {
			save_node = to_rbnode(remove_node->right);
			remove_node->right = nullptr;
		}
		else {
			save_node = to_rbnode(remove_node->left);
			remove_node->left = nullptr;
		}
		// 由后继节点替代时，需要复制数据
		if (remove_node != node) {
			node->data = remove_node->data;
			node->times = remove_node->times;
		}
		// 和父节点的关系重设
		save_node->parent = remove_node->parent;
		if (remove_node->parent) {
			// remove_node is NOT root
			if (remove_node->parent->left == remove_node) {
				remove_node->parent->left = save_node;
			}
			else {
				remove_node->parent->right = save_node;
			}
		}
		else {
			// 删除根节点时更新根
			*root = save_node;
		}
		// 删除了一个黑节点，部分路径黑高度减一，需要恢复性质
		if(remove_node->color==NODE_COLOR::BLACK)
			redblack_tree_delete_fix(root, save_node);
		// 移除待删除节点
		delete remove_node;
		return true;
	}

	// 工具函数，打印内部节点值、颜色、父节点值
	template<typename T>
	void rbtree_print(redblack_tree_node<T>** root) {
		if (*root==nullptr) {
			std::cout << "rbtree empty" << std::endl;
			return;
		}
		std::cout << "====rbtree start====" << std::endl;
		std::vector<binary_tree_node<T>*> order_list;
		inorder<T>(*root, order_list);
		for (auto& t : order_list) {
			std::cout << t->data << " " << enum_trans(to_rbnode(t)->color) << "  <<  ";
			if (t->parent)
				std::cout << t->parent->data << std::endl;
			else
				std::cout << "root" << std::endl;
		}
		std::cout << "====rbtree  end ====" << std::endl << std::endl;
	}
}
template<typename T>
void rbtree_delete_test(lyc_algorithm::redblack_tree_node<T>**root, const T&data) {
	using namespace lyc_algorithm;
	if (redblack_tree_delete<T>(root, data)) {
		rbtree_print(root);
	}
	else {
		std::cout << data << " not exist in rbtree" << std::endl;
	}
}

template<typename T>
void rbtree_insert_test(lyc_algorithm::redblack_tree_node<T>** root, const T& data) {
	using namespace lyc_algorithm;
	redblack_tree_insert<T>(root, data);
	rbtree_print(root);
}

int main() {
	std::cout << "======tree_examples======" << std::endl;
	using namespace lyc_algorithm;
	redblack_tree_node<int>* root = nullptr;
	rbtree_insert_test(&root, 10);
	rbtree_insert_test(&root, 11);
	rbtree_insert_test(&root, 1);
	rbtree_insert_test(&root, 5);
	rbtree_insert_test(&root, -3);
	rbtree_insert_test(&root, 0);
	rbtree_insert_test(&root, 17);
	rbtree_insert_test(&root, 14);
	rbtree_insert_test(&root, 13);
	rbtree_insert_test(&root, 12);

	//std::cout << "====rbtree_delete====" << std::endl;
	rbtree_delete_test(&root, 233);
	rbtree_delete_test(&root, -3);
	rbtree_delete_test(&root, 10);
	rbtree_delete_test(&root, 11);
	rbtree_delete_test(&root, 1);
	rbtree_delete_test(&root, 5);
	rbtree_delete_test(&root, -3);
	rbtree_delete_test(&root, 0);
	rbtree_delete_test(&root, 17);
	rbtree_delete_test(&root, 14);
	rbtree_delete_test(&root, 13);
	rbtree_delete_test(&root, 12);

	rbtree_insert_test(&root, 4);
	rbtree_insert_test(&root, 3);
	rbtree_insert_test(&root, 2);
	rbtree_insert_test(&root, 4);

	rbtree_delete_test(&root, 3);
	rbtree_delete_test(&root, 4);
	rbtree_delete_test(&root, 4);


	redblack_tree_node<std::string>* strRoot = nullptr;
	rbtree_insert_test(&strRoot, std::string("233"));
	rbtree_insert_test(&strRoot, std::string("666"));
	rbtree_insert_test(&strRoot, std::string("789"));
    return 0;
}
```
# 跳跃表
1. 概述：一种基于链表的，平衡、动态的搜索数据结构。相比于B树、红黑树、树堆等复杂的数据结构，跳跃表非常易于实现，而且非常容易理解和记忆。
	- 任何操作的期望时间复杂度均为：$O(logn)$，而且其方差可以控制到非常小
![跳跃树示意图](/images/algoSeries/skiplist.svg)
2. 操作过程：
	1. 查询：
		1. 从最高层的链表开始，逐个查找直到即将超过范围，进入下一级链表
		2. 重复1，直到找到元素，或在最底层超出范围（即不存在该元素）
	2. 插入：
		- 利用查询寻找该元素：如果找到该元素，增加计数；否则在对应位置插入节点，并计算随机数，以1/2概率向上提升
			- 图中6即为提升2次，1和13各提升一次
	3. 删除：
		- 查询该元素，如果查找到，直接将该节点及其提升节点全部删除。
```cpp
#include<chrono>
#include<random>
#include<iostream>
#include<vector>
namespace lyc_algorithm {

	template<typename T>
	struct skip_node {
		skip_node* left, * right;
		// 为了实现方便，这里和图中是相反的
		// upper代表节点更密集的一层，lower代表节点更稀疏的一层
		skip_node* upper, * lower;
		T value;
		std::size_t times;

		skip_node(const T& data) 
			:value(data), left(nullptr), right(nullptr)
			, upper(nullptr), lower(nullptr),times(1)
		{}

		skip_node()
			:left(nullptr),right(nullptr)
			,upper(nullptr),lower(nullptr),times(0)
		{}
	};

	template<typename T>
	struct skip_list {
		unsigned int max_level;
		std::vector<skip_node<T>*> level;
		skip_node<T>* tail_pointer;

		skip_list(unsigned int max_level = 32)
			:max_level(max_level),tail_pointer(nullptr)
		{
			if (max_level < 1) {
				throw std::exception("max_level must >= 1");
			}
			make_new_level();
		}

		~skip_list() {
			for (auto& node : level) {
				auto save_node = node;
				while (node) {
					save_node = node->right;
					delete node;
					node = save_node;
				}
			}
		}

		// 创建一个新的空层
		void make_new_level() {
			level.push_back(new skip_node<T>());
			auto tempnode = new skip_node<T>();
			level.back()->right = tempnode;
			tempnode->left = level.back();
			if (level.size() > 1) {
				level.back()->upper = level.at(level.size() - 2);
				level.back()->upper->lower = level.back();
				tempnode->upper = tail_pointer;
				tail_pointer->lower = tempnode;
			}
			tail_pointer = tempnode;
		}

		/*
		* 在corner右侧，创建一个新的值节点
		* 其上层节点为*upper_node
		* 产生随机值new_level指示是否继续提升
		*/ 
		void make_level_node(skip_node<T>* corner, const T& data
				, skip_node<T>** upper_node, bool& new_level) {
			static std::uniform_real_distribution<> dis(0, 1);
			auto seed = 
				std::mt19937(std::chrono::system_clock::now().time_since_epoch().count());
			auto old_right = corner->right;
			corner->right = new skip_node<T>(data);
			old_right->left = corner->right;
			corner->right->right = old_right;
			corner->right->left = corner;
			if (*upper_node) {
				corner->right->upper = *upper_node;
				(*upper_node)->lower = corner->right;
			}
			*upper_node = corner->right;
			new_level= dis(seed) < 0.5;
		}

		void insert(const T& data) {
			std::vector<skip_node<T>*> level_corner;
			auto pos = lower_bound(data, level_corner);
			if (pos->times&&pos->value == data) {
				pos->times++;
			}
			else {
				skip_node<T>* upper_node = nullptr;
				bool new_level = false;
				// corner列表是自底向上的，提升是自顶向下的，需要反转一次
				std::reverse(level_corner.begin(), level_corner.end());
				for (auto& corner : level_corner) {
					make_level_node(corner, data, &upper_node, new_level);
					if (!new_level)
						break;
				}
				// 从此层开始是全新的层
				while (new_level && level.size() < max_level) {
					make_new_level();
					make_level_node(level.back(), data, &upper_node, new_level);
				}
			}
		}

		/*
		* 不存在返回false
		* 否则删除并返回 true
		*/
		bool remove(const T& data) {
			auto node = contain(data);
			if (node && node->times > 1) {
				node->times--;
				return true;
			}
			else if (node) {
				while (node) {
					auto lower_node = node->lower;
					node->left->right = node->right;
					node->right->left = node->left;
					delete node;
					node = lower_node;
				}
				// 检查是否有多余的空层
				if (level.size() > 1) {
					// 注意这里不能用迭代器哦，因为涉及到删除操作，迭代器会失效的
					for (size_t i = level.size()-1
							; i != 0 && level.at(i)->right->times == 0
							; --i) {
						tail_pointer = level.at(i)->right->upper;
						delete level.at(i)->right;
						delete level.at(i);
						level.pop_back();
					}
				}
				return true;
			}
			else {
				return false;
			}
		}

		/*
		* 存在返回对应值
		* 否则返回nullptr
		*/
		skip_node<T>* contain(const T& data) {
			auto node = lower_bound(data);
			if (node->times && node->value == data) {
				return node;
			}
			else {
				return nullptr;
			}
		}

		/*
		* 返回第一个大于等于data的值
		* 不存在则返回最底层的哨兵
		*/
		skip_node<T>* lower_bound(const T& data) {
			std::vector<skip_node<T>*> level_corner;
			return lower_bound(data, level_corner);
		}

		/*
		* 返回第一个大于等于data的值
		* 同时返回查询时自底到上每一层转角值
		* 不存在则返回最底层的哨兵
		*/
		skip_node<T>* lower_bound(const T& data, std::vector<skip_node<T>*>& level_corner) {
			skip_node<T>* current_node = level.back();
			while (current_node) {
				// 每一次循环开始时current_node一定位于不可能满足大于等于data的位置
				current_node = current_node->right;
				if (current_node->times) {
					//循环向右找到第一个大于等于的情况
					while (current_node->times && current_node->value < data) {
						current_node = current_node->right;
					}
					//存在等于
					if (current_node->times && current_node->value == data) {
						while (current_node->upper) {
							current_node = current_node->upper;
						}
						return current_node;
					}
					else {//大于或不存在
						//存在上层，继续查询
						if (current_node->upper) {
							level_corner.push_back(current_node->left);
							current_node = current_node->left->upper;
						}
						else {
							//不存在上层了，确实没有该值，直接返回
							level_corner.push_back(current_node->left);
							return current_node;
						}
					}
				}
				else {
					// 该层到哨兵了，该层不存在大于等于
					level_corner.push_back(current_node->left);
					// 可以向上
					if (current_node->upper)
						current_node = current_node->left->upper;
					else
						return current_node;
				}
			}
		}
	};

	template<typename T>
	std::ostream& operator<<(std::ostream& o, const skip_list<T>& list) {
		for (auto& level : list.level) {
			o << "^ ";
			auto node = level->right;
			while (node) {
				if (node->times) {
					o << " <-> " << node->value;
				}
				else {
					o << " <-> $";
				}
				node = node->right;
			}
			o << std::endl;
		}
		return o;
	}
}
int main(){
	using namespace lyc_algorithm;
	skip_list<int> int_skip_list;
	int_skip_list.insert(1);
	int_skip_list.insert(2);
	int_skip_list.insert(7);
	int_skip_list.insert(11);
	int_skip_list.insert(-7);
	int_skip_list.insert(0);
	int_skip_list.insert(7);
	int_skip_list.insert(1);
	int_skip_list.insert(100);
	int_skip_list.insert(6);
	std::cout << int_skip_list << std::endl;
	std::cout << "search 6: " << int_skip_list.contain(6)->value << std::endl;
	std::cout << "search 100: " << int_skip_list.contain(100)->value << std::endl;
	std::cout << "search -7: " << int_skip_list.contain(-7)->value << std::endl;
	std::cout << "remove 100: " << int_skip_list.remove(100) << std::endl;
	std::cout << "remove 1: " << int_skip_list.remove(1) << std::endl;
	std::cout << "remove 2: " << int_skip_list.remove(2) << std::endl;
	std::cout << "remove 7: " << int_skip_list.remove(7) << std::endl;
	std::cout << "remove 11: " << int_skip_list.remove(11) << std::endl;
	std::cout << "remove -1: " << int_skip_list.remove(-7) << std::endl;
	std::cout << int_skip_list << std::endl;
	std::cout << "remove 1: " << int_skip_list.remove(1) << std::endl;
	std::cout << "remove 7: " << int_skip_list.remove(7) << std::endl;
	std::cout << "remove 6: " << int_skip_list.remove(6) << std::endl;
	std::cout << int_skip_list << std::endl;
	std::cout << "remove 0: " << int_skip_list.remove(0) << std::endl;
	std::cout << int_skip_list << std::endl;
	return 0;
}
```
# B树
- 出现背景：B树的最初出现是为了对磁盘等辅助存储设备上的数据存储而设计的一类平衡搜索树，合理使用B树能够降低磁盘IO次数。虽然目前SSD已经开始大规模普及，但考虑到价格和稳定性工业界仍然需要大量HDD的存在。
	- 简而言之，B树的每一个节点聚合了大量的关键字，比如1000个，那么一个高度为2的B树，实际已经可以存储10亿量级的关键字，而且存取任意关键字，只需要两次读盘。
	- 实现上，一个节点可以恰好构造为一个虚拟文件系统的一个页。
	- 根节点一般常驻内存。
	- 假定没有重复元素（大多数树都以这种条件要求，不过实现支持重复元素也很简单就是了）
	- 稳定的渐进时间复杂度$O(logn)$。
- 定义：一颗B树是具有以下性质的有根树
	- 每个节点$x$包含域：$n\[x\]$(当前存储在节点x中的关键字数量)，$key_1\[x\]...key_{n\[x\]}\[x\]$（全部关键字，按非降序排列），$leaf\[x\]$（指示节点x是否为叶子节点的布尔值）。
	- 指向子女的指针：$c_1\[x\]...c_{n\[x\]+1}\[x\]$，根据全部关键字划分，恰好为$n\[x\]+1$个。
	- 每个叶子节点具有相同的深度，等于树的高度
	- 每一个节点包含的关键字数量有上界和下界。以称$t$为B树的最小度数，则有：
		- 每个非根节点必须至少有$t-1$个关键字，至少有$t$个子女。如果树非空，根节点至少有$1$个关键字。
		- 每个节点至多有$2t-1$个关键字，即至多有$2t$个子女。如果达到此最大值，称该节点是满的。
		> $t=2$时的B树最为简单，也就构成了一个2-3-4树。而红黑树和2-3-4树也有一定的关系。由此可以看出，B树、2-3-4树、红黑树，都有着一定的联系。

		> 关于度。也有说法是阶。来自于Wikipedia的说法是，一个m阶的B树，每一个节点最多有m个子节点，每一个非根内部节点最少有$\lceil m/2 \rceil$。由于和算法导论有些出入。因此本文的描述仍然以算法导论为主。
- 基本操作：由于B树的出现和磁盘操作非常相关，因此这里也保留了读盘$\mathrm{DISK\\_READ}()$、写盘$\mathrm{DISK\\_WRITE}()$操作。
	1. 创建新节点：$\mathrm{ALLOCATE\\_NODE}()$
	2. 创建空树：$\mathrm{B\\_TREE\\_CREATE}(T)$
		1. &emsp;$\mathrm{ALLOCATE\\_NODE}(x)$
		2. &emsp;$leaf\[x\] \gets true$
		3. &emsp;$n\[x\] \gets 0$
		4. &emsp;$DISK\\_WRITE(x)$
		5. &emsp;$root[T]\gets x$
	3. 搜索：$\mathrm{B\\_TREE\\_SEARCH}(x,k)$
		1. &emsp;$i \gets 1$
		2. &emsp;$\mathbf{while} \ i \le n\[x\] \ and \ k > key_i\[x\]$
		3. &emsp;&emsp;&emsp; $\mathbf{do} \ i \gets i+1$
		4. &emsp;$\mathbf{if} \ i \le n\[x\] \ and \ k=key_i\[x\]$
		5. &emsp;&emsp;&emsp; $\mathbf{then \ return}(x,i)$
		6. &emsp;$\mathbf{if} \ leaf\[x\]$
		7. &emsp;&emsp;&emsp; $\mathbf{then \ return} \ \mathrm{NIL}$
		8. &emsp;$\mathbf{else}$
		9. &emsp;&emsp;&emsp; $\mathrm{DISK\\_READ}(c_i\[x\])$
		9. &emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_SEARCH}(c_i\[x\],k)$
	4. 插入：B_TREE_INSERT($T,k$)

		![B树节点分裂](/images/algoSeries/BTreeSplitNode.svg)
		
		> 先来思考一下插入复杂的原因哈。我们找到位置之后，如果不满，那直接插入了。否则，该节点需要分裂。更麻烦的是，该节点分裂之后，可能父节点超出了满节点的限制，父节点还需要分裂，以此向上直到根节点，都需要处理。
		- 满节点分裂子程序：$\mathrm{B\\_TREE\\_SPLIT\\_CHILD}(x,i,y)$，其中$y=c_i\[x\]$是$x$的一个满子节点。
			1. &emsp;$z \gets \mathrm{ALLOCATE\\_NODE}()$
			2. &emsp;$leaf[z] \gets leaf[y]$
			3. &emsp;$n[z] \gets t-1$
			4. &emsp;$\mathbf{for} \ j \gets \mathbf{to} \ t-1$
			5. &emsp;&emsp;&emsp; $\mathbf{do} \ key_i[z] \gets key_{j+t}[y]$
			6. &emsp;$\mathbf{if} \ \mathrm{not} \ leaf[y]$
			7. &emsp;&emsp;&emsp; $\mathbf{then} \ \mathbf{for} \ j \gets 1 \ \mathbf{to} \ t$
			8. &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $\mathbf{do} \ c_j[z] \gets c_{j+t}[y]$
			9. &emsp; $n[y] \gets t-1$
			10. &emsp; $\mathbf{for} \ j \gets n\[x\] + 1 \ \mathbf{downto} \ i + 1 $
			11. &emsp;&emsp;&emsp; $\mathbf{do} \ c_{j+1}\[x\] \gets c_j\[x\]$
			12. &emsp; $key_i\[x\] \gets key_t\[x\]$
			13. &emsp; $n\[x\] \gets n\[x\] + 1$
			14. &emsp; $\mathrm{DISK\\_WRITE}(y)$
			15. &emsp; $\mathrm{DISK\\_WRITE}(z)$
			16. &emsp; $\mathrm{DISK\\_WRITE}(x)$
			> 简单总结：由于满节点关键字一定是奇数个。分裂将满节点的中间的关键字提升到父节点，其余关键字一半一半。
		- 非满节点插入子程序：$\mathrm{B\\_TREE\\_INSERT\\_NONFULL}(x,k)$
			1. &emsp; $i \gets n\[x\]$
			1. &emsp; $\mathbf{if} \ leaf\[x\]$
			1. &emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp; $\mathbf{while} \ i \ge 1 \ \mathrm{and} \ k < key_i\[x\]$
			1. &emsp;&emsp;&emsp; $\mathbf{do}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $key_{i+1}\[x\] \gets key_i\[x\]$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $i \gets i-1$
			1. &emsp;&emsp;&emsp; $key_{i+1}\[x\] \gets k$
			1. &emsp;&emsp;&emsp; $n\[x\] \gets n\[x\]+1$
			1. &emsp;&emsp;&emsp; $\mathrm{DISK\\_WRITE}(x)$
			1. &emsp; $\mathbf{else}$
			1. &emsp;&emsp;&emsp; $\mathbf{while} \ i \ge 1 \ and \ k < key_i\[x\]$
			1. &emsp;&emsp;&emsp; $\mathbf{do}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $i \gets i-1$
			1. &emsp;&emsp;&emsp; $i \gets i+1$
			1. &emsp;&emsp;&emsp; $\mathrm{DISK\\_READ}(c_i\[x\])$
			1. &emsp;&emsp;&emsp; $\mathbf{if} \ n[c_i\[x\]]  = 2t-1$
			1. &emsp;&emsp;&emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_SPLIT\\_CHILD}(x,i,c_i\[x\])$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathbf{if} \ k > key_i\[x\]$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $i \gets i+1$
			1. &emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_INSERT\\_NONFULL}(c_i\[x\],k)$
			> 简单总结：非满叶子节点直接插入即可；而非满内节点，需要查找合适关键字位置，如果发现其位于满节点，则进行一次分裂（算法保证当前节点非满，分裂一定可以进行），并继续向下遍历子树。
		- 节点插入：$\mathrm{B\\_TREE\\_INSERT}(T,k)$
			1. &emsp; $r \gets root[T]$
			2. &emsp; $\mathbf{if} \ n[r]=2t-1$
			3. &emsp; $\mathbf{then}$ 
			4. &emsp;&emsp;&emsp; $s \gets \mathrm{ALLOCATE\\_NODE}()$
			4. &emsp;&emsp;&emsp; $root[T] \gets s$
			5. &emsp;&emsp;&emsp; $leaf[s] \gets \mathrm{FALSE}$
			6. &emsp;&emsp;&emsp; $n[s] \gets 0$
			7. &emsp;&emsp;&emsp; $c_1\[x\] \gets r$
			8. &emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_SPLIT\\_CHILD}(s,1,r)$
			9. &emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_INSERT\\_NONFULL}(s,k)$
			10. &emsp; $\mathbf{else}$
			12. &emsp;&emsp;&emsp; $\mathrm{B\\_TREE\\_INSERT\\_NONFULL}(r,k)$
			> B树的操作推崇提高磁盘效率（不要回溯）。因此插入的过程是单程下行遍历树的。这也代表节点的分裂是从根开始的，自顶向下进行。
	4. 删除：$\mathrm{B\\_TREE\\_DELETE}(x,k)$
		> 删除操作与插入类似，但是更为复杂。从插入操作的非满节点插入子程序可以发现，新节点的插入一定是在叶子节点中进行的。但删除操作则不是，删除可以发生在任何位置。
		- 整体保证：对节点$x$递归向下调用（注意是说如果发生了向下调用）$\mathrm{B\\_TREE\\_DELETE}(x,k)$后，$x$的关键字个数都至少等于最小度数$t$（而不是$t-1$)。这种保证能够提高效率，减少回溯。

		![BTree删除节点的情况示例](/images/algoSeries/BTreeDelete.svg)
		<center>$t=3$的BTree的删除示例</center></br>
		
		1. 情况一：如果关键字在节点$x$中，而且$x$是个叶子节点，则从$x$中删除k。
		2. 情况二：如果关键字在节点$x$中，而且$x$是个内节点，则再分以下情况讨论：
			1. 情况A：如果节点$x$内，$k$和$k$前面的关键字之间的子节点$y$包含至少$t$个关键字，则找出$y$为根的子树中的$k$的前驱$k'$，删除$k'$，并在$x$中，用$k'$替代$k$。
			2. 情况B：对称的，如果节点$x$内，$k$和$k$后面的关键字之间的子节点$y$包含至少$t$个关键字，则找出$y$为根的子树中的$k$的后继$k'$，删除$k'$，并在$x$中，用$k'$替代$k$。
			3. 情况C：否则，意味着$k$前后关键字之间的子节点的关键字数量均小于等于$t-1$个，将这两个子节点合并为新节点$z$，删除$k$，并修改子节点指针指向$z$。
		3. 情况三：如果关键字不在当前内结点$x$中，则先确定包含$k$的子树的根$c_i\[x\]$。如果$c_i\[x\]$只有$t-1$个关键字，分以下情况讨论之后再递归下降到一个合适的节点进行删除。
			1. 情况A：如果$c_i\[x\]$只包含$t-1$个关键字，但它的一个相邻兄弟包含至少$t$个关键字，则将$x$中的某一个关键字下降至$c_i\[x\]$中，将$c_i\[x\]$相邻的兄弟节点中的某一个关键字升至$x$，将该兄弟中合适的子女指针移动到$c_i\[x\]$中。
			2. 情况B：如果$c_i\[x\]$以及它的所有相邻兄弟都只有$t-1$个关键字，则将$c_i\[x\]$和一个兄弟合并，并将$x$的一个关键字移动至新合并的节点，并使之成为改节点正中间的关键字。
			3. 经过A、B处理后，调用$\mathrm{B\\_TREE\\_DELETE}(c_i\[x\],k)$
		- 说明：各种情况ABC，都是为了防止删除导致出现过小的$n\[x\]$，保证B树依然维持原有的性质。结合图例，注意理解：删除$F$时不需要将$CGMTX$合并，是因为$DEF$并没有再递归下降，函数执行到这里，删除掉$F$就返回了。而删除$D$的时候就正好满足情况三的要求，而且触发了删除根$P$。也就是正确理解三种情况的考虑顺序。只有当需要出现递归下降时，才需要进行一次情况三的考虑。
- 实际应用：MySQL等数据库的存储引擎，以及其他各类应用中，实际更经常使用的是B树的变种B+树、B*树。
	- B+树定义（和B树不同的）：
		1. 数据仅存在于叶节点，内部节点仅用于检索。（内存可以容纳更多的索引）
		2. 叶子节点的兄弟节点之间存在指针相连。（更便于对节点的顺序遍历）
		3. 内部节点的孩子指针数量和关键字数量相同。$c_i\[x\]$子树范围为$[Key_i\[x\],Key_{i+1}\[x\])$，作为对比，B-Tree则是开区间。
		4. 分裂操作：当一个节点满时，分配新节点，并将原节点中一般的数据复制到新节点。
	- B*树定义（和B+树不同的）：
		1. 非根内部节点的兄弟之间也有指针连接。
		2. 分裂操作：当一个节点满时，如果递增方向的兄弟节点未满，则转移一部分给兄弟节点，并修改父节点中的对应关键字，如果递增方向的兄弟节点也满了，则在二者之间新增节点，并各复制$1/3$的数据给新节点，父节点增加关键字。（非根节点的最低利用率由$1/2$上升到$2/3$）
# 树堆（Treap）
- 定义： 每一个节点除了关键字以外，还有一个随机值作为优先级。使得整棵树不仅在关键字上满足二叉搜索树的定义，同时在该随机优先级值上满足大顶堆（小顶堆也行）的性质。即树（Tree）和堆（Heap）的组合，树堆（Treap）。
- 性能：期望时间复杂度$O(logn)$。
	- 相较于其他平衡二叉搜索树，实现相对更为简单，且基本能实现随机平衡的结构。
- 操作：
	1. 插入：
		1. 将新关键字插入到满足二叉搜索树要求的位置
		2. 赋予新节点随机值作为优先级
		3. 从新节点自底向上递归，使用左旋或右旋使针对优先级满足大顶堆性质
	2. 删除：
		1. 找到待删除节点
		2. 将该节点和子节点比较，向下进行左旋或者右旋
		3. 循环进行步骤2，直到该节点变换到叶子节点，删除
	3. 查找：和普通二叉搜索树一致
# 可合并堆
- 可合并堆：支持如下操作的数据结构：
	- 创建并返回空的新堆
	- 向堆中插入关键字
	- 返回指向堆中最小关键字节点的指针
	- 将堆中包含最小关键字的节点从堆中删除，返回其指针
	- 将两个堆合并
	> 默认的可合并堆都是最小堆
- 性能对比
|  操作   | 二叉堆（最坏情况） | 二项堆（最坏情况） | 斐波那契堆（均摊） |
|  ----  | ----  |  ----  | ----  |  ----  |
|  建堆 | $\Theta(1)$ | $\Theta(1)$ | $\Theta(1)$ |
|  插入 | $\Theta(logn)$ | $\Omega(logn)$ | $\Theta(1)$ |
|  返回最小元素 | $\Theta(1)$ | $\Omega(logn)$ | $\\Theta(1)$ |
|  抽取最小元素 | $\Theta(logn)$ | $\Theta(logn)$ | $\mathrm{O}(logn)$ |
|  合并堆 | $\Theta(n)$ | $\Omega(logn)$ | $\Theta(1)$ |
|  减小关键字 | $\Theta(logn)$ | $\Theta(logn)$ | $\Theta(1)$ |
|  删除关键字 | $\Theta(logn)$ | $\Theta(logn)$ | $\mathrm{O}(logn)$ |
- 图例
![二项堆和斐波那契堆示意图](/images/algoSeries/binomailHeap_fiboHeap.svg)
<center>二项堆(上)和斐波那契堆(下)示意图。注：红色代表标记</center>

# 二项堆
- 定义
	- 二项树：一种递归定义的有序树
		- 二项树$B_0$只包含一个节点
		- 二项树$B_k$由两棵二项树$B_{k-1}$连接而成，其中一棵树的根是另一棵树的根的最左孩子
	- 二项树$B_k$的性质：
		- 共有$2^k$个节点
		- 树的高度为k
		- 在深度$i$处，恰好有$\binom{k}{i}$个节点。$i=0,1,2,...k$
		- 根的度数为$k$，是树中的最大度数，并且根的子女从左至右依次是二项树$B_{k-1},B_{k-2}...B_0$的根。
		- 二项树是左偏的（树中的任何一个子树，左侧子树节点多于右侧兄弟子树节点）
	- 二项堆：由满足以下性质的二项树组成
		- 二项堆中的每个二项树遵循最小堆性质，节点的关键字不小于父节点关键字。即二项树是最小堆有序的。
		- 对任意非负整数$k$，在二项堆中至多有一棵二项树的根具有度数$k$。
			- 即一个包含$n$个节点的二项堆中，包含至多$\lfloor logn \rfloor +1$棵二项树。关于这一点的理解，可以参考二进制编码。数字$n$的二进制编码有$\lfloor logn \rfloor +1$位，记作$< b_{\lfloor logn \rfloor},b_{\lfloor logn \rfloor -1},...,b_0 >$，有$n=\sum_{i=0}^{\lfloor logn \rfloor}b_i2^i$，恰好可将和式中的每一项视为一个二项树。
	- 二项堆的表示：
		- 每个二项树节点$x$的域：
			- 父节点指针：$p\[x\]$
			- 最左孩子的指针：$child\[x\]$
			- 指向紧右兄弟的指针：$sibling\[x\]$
			- 子女数量（度）：$degree\[x\]$
		- 二项堆$H$的表示：
			- 首个二项树的根：$head[H]$
			- 根表（包含各二项树的根的链表）：$sibling\[x\]$
				- 就是各根节点的紧右兄弟的指针
				- 根表中各根的度数严格递增（和非根节点的$sibling\[x\]$域的特点相反）
	- 在可合并堆基础上的额外操作：
		- 减少关键字的大小
		- 删除某个关键字
- 操作：
	1. 创建新堆：$\mathrm{MAKE\\_BINOMAIL\\_HEAP}()$
	2. 寻找最小关键字：$\mathrm{BINOMAIL\\_HEAP\\_MINIMUM}(H)$
		- 从$head[H]$开始，沿着根表寻找最少值
	3. 合并两个二项堆：$\mathrm{BINOMAIL\\_HEAP\\_UNION}(H_1,H_2)$
		- 两棵$B_{k-1}$树的合并子程序：$\mathrm{BINOMAIL\\_LINK}(y,z)$
			1. &emsp; $p[y] \gets z$
			1. &emsp; $sibling[y] \gets child[z]$
			1. &emsp; $child[z] \gets y$
			1. &emsp; $degree[z] \gets degree[z] + 1$
			> $y$成为了$z$的最左孩子
		
		- 根表合并子程序：$\mathrm{BINOMAIL\\_HEAP\\_MERGE}(H_1,H_2)$
			1. &emsp; $x \gets head[H_1]$
			1. &emsp; $y \gets head[H_2]$
			1. &emsp; $root \gets \mathrm{NIL}$
			1. &emsp; $current \gets \\&root$
			1. &emsp; $\mathbf{while} \ x \ne \mathrm{NIL \ and} \ y \ne \mathrm{NIL}$
			1. &emsp; $\mathbf{do}$
			1. &emsp;&emsp;&emsp; $\mathbf{if} \ degree\[x\] < degree[y]$
			1. &emsp;&emsp;&emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $sibling[*current] \gets x$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $x \gets sibling\[x\]$
			1. &emsp;&emsp;&emsp; $\mathbf{else}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $sibling[*current] \gets y$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $y \gets sibling[y]$
			1. &emsp;&emsp;&emsp; $current \gets \\&sibling[*current]$
			1. &emsp; $\mathbf{if} x \ne \mathrm{NIL}$
			1. &emsp;&emsp;&emsp; $sibling[*current] \gets x$
			1. &emsp; $\mathbf{else}$
			1. &emsp;&emsp;&emsp; $sibling[*current] \gets y$
			1. &emsp; $\mathbf{return} \ root$
			> 注：1.伪代码中的*和&为C/C++语法中的取地址和解引用运算。2.根表合并只保证根表中的度数单调不递减。

		- 二项堆合并程序：$\mathrm{BINOMAIL\\_HEAP\\_UNION}(H_1,H_2)$
			1. &emsp; $H \gets \mathrm{MAKE\\_BINOMAIL\\_HEAP}()$
			1. &emsp; $head[H] \gets \mathrm{BINOMAIL\\_HEAP\\_MERGE}(H_1,H_2)$
			1. &emsp; $\mathbf{if} \ head[H] = \mathrm{NIL}$
			1. &emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp; $\mathbf{return} \ H$
			1. &emsp; $prev\\_x \gets NIL$
			1. &emsp; $x \gets head[H]$
			1. &emsp; $next\\_x \gets sibling\[x\]$
			1. &emsp; $\mathbf{while} \ next\\_x \ne \mathrm{NIL}$
			1. &emsp; $\mathbf{do}$
			1. &emsp;&emsp;&emsp; $\mathbf{if} (degree\[x\] \ne degree[next\\_x])$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathrm{or} \ (sibling[next\\_x] \ne \mathrm{NIL} \ \mathrm{and}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $degree[sibling[next\\_x]]=degree\[x\])$
			1. &emsp;&emsp;&emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $prev\\_x \gets x$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $x \gets next\\_x$
			1. &emsp;&emsp;&emsp; $\mathbf{else \ if} \ key\[x\] \le key[next\\_x]$
			1. &emsp;&emsp;&emsp; $\mathbf{then}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $sibling\[x\] \gets sibling[next\\_x]$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathrm{BINOMAIL\\_LINK}(next\\_x,x)$
			1. &emsp;&emsp;&emsp; $\mathbf{else}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathbf{if} \ prev\\_x = \mathrm{NIL}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  $head[H] \gets next\\_x$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathbf{else}$
			1. &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $sibling[prev\\_x] \gets next\\_x$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $\mathrm{BINOMAIL\\_LINK}(x,next\\_x)$
			1. &emsp;&emsp;&emsp;&emsp;&emsp; $x \gets next\\_x$
			1. &emsp;&emsp;&emsp; $next\\_x \gets sibling\[x\]$
			1. &emsp; $\mathbf{return} \ H$
			> 情况说明
			1. 情况一（第14行）：虽然刚合并的根表不会出现多于2个的重复度数的根，但是在逐渐合并根的过程中，会出现至多3个重复度数的根。此时第一个根被保留，第二个第三个后续进行合并。
			2. 情况二（第17行）：合并根节点需要满足最小堆性质
			3. 情况三（第22行）：比较大小后，根据需要更换二项堆的根节点，或者调整前后的兄弟关系，最后合并根节点。

	4. 插入一个节点：$\mathrm{BINOMAIL\\_HEAP\\_INSERT}(H,x)$
		1. &emsp; $H' \gets \mathrm{MAKE\\_BINOMAIL\\_HEAP}()$
		1. &emsp; $p\[x\] \gets \mathrm{NIL}$
		1. &emsp; $child\[x\] \gets \mathrm{NIL}$
		1. &emsp; $sibling\[x\] \gets \mathrm{NIL}$
		1. &emsp; $degree\[x\] \gets 0$
		1. &emsp; $head[H'] \gets x$
		1. &emsp; $H \gets \mathrm{BINOMAIL\\_HEAP\\_UNION}(H,H')$
		> 插入节点等价于：创建一个新的单点构成的堆，并和已有堆合并
	
	5. 抽取具有最小关键字的节点：$\mathrm{BINOMAIL\\_HEAP\\_EXTRACT\\_MIN}(H)$
		1. &emsp; 找到最小关键字节点（必在二项堆根表中）$x$
		1. &emsp; 从根表中移除$x$
		1. &emsp; $H' \gets \mathrm{MAKE\\_BINOMAIL\\_HEAP}()$
		1. &emsp; 反转$x$的孩子节点的链表，记为$head\\_child\\_x$
		1. &emsp; $head[H'] \gets head\\_child\\_x$
		1. &emsp; $H \gets \mathrm{BINOMAIL\\_HEAP\\_UNION}(H,H')$
		1. &emsp; $\mathbf{return} \ x$
		> 其实就是通过反转一次链表，将$x$的孩子们恰好组成一个二项堆，并和已有二项堆合并

	6. 将关键字的值由$x$减小为$k$：$\mathrm{BINOMAIL\\_HEAP\\_DECREASE\\_KEY}(H,x,k)$
		1. &emsp; $\mathbf{if} \ k > key\[x\]$
		1. &emsp;&emsp;&emsp; $\mathbf{error}$
		1. &emsp; $key\[x\] \gets k$
		1. &emsp; $y \gets x$
		1. &emsp; $z \gets p[y]$
		1. &emsp; $\mathbf{while} \ z \ne \mathrm{NIL} \ and \ key[y] < key[z]$
		1. &emsp; $\mathbf{do}$
		1. &emsp;&emsp;&emsp; $exchange \ key[y] \leftrightarrow key[z]$
		1. &emsp;&emsp;&emsp; 如果y、z有卫星数据，一并交换
		1. &emsp;&emsp;&emsp; $y \gets z$
		1. &emsp;&emsp;&emsp; $z \gets p[y]$
		> 关键字减小，只需要自底向上，不断和父节点比较大小并交换即可。

	7. 删除一个关键字：$\mathrm{BINOMAIL\\_HEAP\\_DELETE}(H,x)$
		1. &emsp; $\mathrm{BINOMAIL\\_HEAP\\_DECREASE\\_KEY}(H,x,-\infty)$
		1. &emsp; $\mathrm{BINOMAIL\\_HEAP\\_EXTRACT\\_MIN}(H)$
		> 删除关键字相当于，先把该关键字变为无穷小（称为最小元素），再从二项堆中提取最小元素。

# 斐波那契堆
- 也是可合并堆的一种，提供可合并堆的各类操作。为了改善时间夫再度，斐波那契堆以均摊分析为指导，维护一个稍显宽松的二项堆。理论上，使用斐波那契堆作为基础数据结构能够改善很多算法（但实际常数因子可能稍高）。
- 定义：
	- 由一组最小堆有序树构成，但堆中的树并不一定是二项树。
	- 每一个节点的域：
		- 指向父节点：$p\[x\]$
		- 指向任意一个子女：$child\[x\]$
		- 指向前后兄弟：$left\[x\],right\[x\]$
		- 度数：$degree\[x\]$
		- 标记：$mark\[x\]$指示$x$从上一次成为某个节点的子女之后，是否失去过一个子女节点。
	- 堆的域：
		- 指向包含最小关键字的树根：$min[H]$
		- 堆中节点个数：$n[H]$
	- 斐波那契堆中的树是无序的（即各数之间没有度数单调的要求，且同一层的子树也没有度数单调的要求）
- 分析：
	- 根表中的树的数量：$t(H)$
	- 堆中有标记节点的数量：$m(H)$
	- 势函数：
		$\Phi(H)=t(H)+2m(H)$
	- 最大度数的上界：$D(n)$
		- 当斐波那契堆只需支持可合并堆的操作时：$D(n) \le \lfloor logn \rfloor$
		- 当斐波那契堆还需要支持减少关键字和删除关键字时：$D(n) = O(logn)$
- 操作的复杂度分析
	1. 创建新堆：
		- 流程：直接创建
		- 势函数变化：无，均摊代价为$O(1)$
	1. 插入节点：
		- 流程：直接将节点插入根表
		- 势函数变化：$\Delta\Phi(H)=((t(H)+1)+2m(H))-(t(H)+2m(H))=1$，均摊代价仍为$O(1)$
	1. 寻找最小节点：
		- 流程：直接返回$min[H]$
		- 势函数变化：无，均摊代价为$O(1)$
	1. 合并两个斐波那契堆：
		- 流程：直接链接两个堆的根表，保留最小的$min[H]$作为新的根
		- 势函数变化：$\Delta\Phi(H)=\Phi(H)-(\Phi(H_1)+\Phi(H_2))=0$，均摊代价仍为$O(1)$。（本质上两个堆合并，其根表中树的总量、有标记节点的数量都没有变化。）
	1. 抽取最小节点：
		- 流程：从跟表中断开$min[H]$，将$min[H]$原子节点层提升到根表内。进行一次consolidate操作（从任意根表结点开始，合并根表节点称为最小堆无序树，最终使新根表中不再有重复度数的根）
		- 势函数变化：$\Delta\Phi(H) = ((D(n)+1)+2m(H))-(t(H)+2m(H)) = O(t(H))-t(H)$
			- 均摊代价还要考虑其他操作，至多为$O(D(n)+t(H))+\Delta\Phi(H) = O(D(n))+O(t(H))-t(H)=O(D(n))$
		- 详细解释：
			1. 在抽取过程中，首先需要处理$min[H]$的至多$D(n)$个子女，再加上断开根表的$O(1)$，以及consolidate过程$O(?)$，总时间代价为$O(D(n)+?)$。
			2. 仔细分析consolidate过程，根表大小此时至多为$D(n)+t(H)-1$。虽然是两层循环，但是每一次内层循环被调用时，根表大小都会下降，因此实际仍为单次扫描的时间。总体时间可视为$O(D(n)+t(H))$。
			3. 综上一次合并的时间代价为$O(D(n)+t(H))$。
			4. 均摊代价最终结果为$O(D(n))$，而由最大度数的上界的性质可知，该值为$O(logn)$。
	1. 减小关键字：
		- 流程：直接修改关键字，如果违反最小堆要求，则和父节点断开链接，当前关键字进入根表，如有需要替换$min[H]$。并对父节点递归向上，检查是否需要级联断开（检查标记）。
		- 势函数变化：假设整个过程中，有$c$个节点被断开，并因此进入根表。则树的数量至多+c，标记的数量至少下降-(c-1)+1（关键字所在节点原来就没标记，最后一个节点增加标记）。此时变化至多为，$\Delta\Phi(H)=((t(H)+c)+2(m(H)-c+2))-(t(H)+2m(H))=4-c。
			- 实际代价：修改关键字、断开链接、进入根表$O(1)$。级联删除$O(c)$。
			- 均摊代价：$O(c)+4-c=O(1)$
	1. 删除节点：
		- 流程：和二项堆一样，减少关键字和抽取最小节点相结合
		- 均摊代价：两者的结合，$O(logn)$
- 代码
```cpp
#include<exception>
#include<vector>
#include<cstdlib>
#include<iostream>
#include<queue>
namespace lyc_algorithm {
	template<typename T>
	struct fibo_heap_node {
		fibo_heap_node* left, * right, * parent, * child;
		size_t degree;
		bool mark;
		T value;
		
		fibo_heap_node()
			:left(nullptr), right(nullptr)
			, parent(nullptr), child(nullptr)
			, mark(false), degree(0) {}

		fibo_heap_node(const T& value, fibo_heap_node* parent=nullptr)
			:left(this), right(this), parent(parent)
			, child(nullptr), value(value)
			, mark(false), degree(0) {

		}
	};

	template<typename T>
	class fibo_heap {
	private:
		fibo_heap_node<T>* min_root;
		size_t n;


		void consolidate() {
			// 临时数组中存储了按度为下标的当前的根表
			std::vector<fibo_heap_node<T>*> root_degree_list(log2(n)+1,nullptr);
			std::vector<fibo_heap_node<T>*> root_list;
			// 由于根表将会变动，因此提前保存
			root_list.push_back(min_root);
			for (auto root_node = min_root->right
				; root_node != min_root
				; root_node = root_node->right) {
				root_list.push_back(root_node);
			}
			for (auto& root_node : root_list) {
				auto current_root = root_node;
				auto degree = current_root->degree;
				// 处理重复度数的根
				for (; root_degree_list[degree];) {
					auto another_root = root_degree_list[degree];
					if (current_root->value>another_root->value) {
						// 交换指针
						std::swap(current_root, another_root);
					}
					// 将较小的作为根
					link(current_root, another_root);
					// 清空该度数的根，继续查找更大的度数是否有重复的根
					root_degree_list[degree] = nullptr;
					degree++;
				}
				root_degree_list[current_root->degree] = current_root;
			}
			min_root = nullptr;
			for (size_t i = 0; i != root_degree_list.size(); ++i) {
				if (root_degree_list[i]){
					// 重建根表
					root_degree_list[i]->parent = nullptr;
					if (!min_root) {
						min_root = root_degree_list[i];
						min_root->left = min_root;
						min_root->right = min_root;
					}
					else {
						auto left = min_root->left;
						left->right = root_degree_list[i];
						root_degree_list[i]->left = left;
						min_root->left = root_degree_list[i];
						root_degree_list[i]->right = min_root;
					}
					// 记录斐波那契堆的根
					if(root_degree_list[i]->value < min_root->value) {
						min_root = root_degree_list[i];
					}
				}
			}
		}

		void link(fibo_heap_node<T>* root, fibo_heap_node<T>* child) {
 			// 将child作为root的子节点
			child->left->right = child->right;
			child->right->left = child->left;
			if (root->child) {
				auto left = root->child->left;
				left->right = child;
				child->left = left;
				child->right = root->child;
				root->child->left = child;
			}
			else {
				root->child = child;
				child->left = child;
				child->right = child;
			}
			child->parent = root;
			child->mark = false;
			root->degree++;
		}

		void cut(fibo_heap_node<T>* child, fibo_heap_node<T>* parent) {
			if (parent->child == child) {
				parent->child = child->right;
			}
			if (parent->child == child) {
				parent->child = nullptr;
			}
			// 调整将删除的子节点的兄弟关系
			child->left->right = child->right;
			child->right->left = child->left;
			// 移动子节点到根表
			child->parent = nullptr;
			child->mark = false;
			auto left = min_root->left;
			left->right = child;
			child->left = left;
			child->right = min_root;
			min_root->left = child;
		}

		void cascade_cut(fibo_heap_node<T>* node) {
			auto parent = node->parent;
			if (parent) {
				if (!parent->mark) {
					parent->mark = true;
				}
				else {
					cut(node, parent);
					cascade_cut(parent);
				}
			}
		}

	public:
		fibo_heap()
			:min_root(nullptr), n(0) {}

		void insert(fibo_heap_node<T>* node) {
			node->left = node;
			node->right = node;
			if (min_root) {
				auto left = min_root->left;
				left->right = node;
				node->left = left;
				node->right = min_root;
				min_root->left = node;
				if (min_root->value > node->value) {
					min_root = node;
				}
			}
			else {
				min_root = node;
			}
			n++;
		}

		void insert(const T& key) {
			auto node = new fibo_heap_node<T>(key);
			insert(node);
		}

		T get_minimum() {
			if (!min_root)
				throw std::exception("empty fibo heap");
			return min_root->value;
		}

		bool empty() {
			return min_root == nullptr;
		}

		void merge(fibo_heap& other) {
			if (other.empty()) {
				return;
			}
			if (!min_root) {
				min_root = other.min_root;
			}
			else {
				auto left = min_root->left;
				auto other_left = other.min_root->left;
				left->right = other.min_root;
				other.min_root->left = left;
				min_root->left = other_left;
				other_left->right = min_root;
				if (min_root->value > other.min_root->value) {
					min_root = other.min_root;
				}
			}
			// release 
			other.min_root = nullptr;
			n += other.n;
			other.n = 0;
		}

		fibo_heap_node<T>* extract_minimun() {
			auto ret = min_root;
			if (ret) {
				// 移动所有子节点到根表
				auto child = ret->child;
				if (child) {
					auto left = ret->left;
					auto child_left = child->left;
					left->right = child;
					child->left = left;
					ret->left = child_left;
					child_left->right = ret;
				}
				// 从根表中删除最小节点
				ret->left->right = ret->right;
				ret->right->left = ret->left;
				// 根据情况调整
				if (n == 1) {
					min_root = nullptr;
				}
				else {
					min_root = ret->left;
					consolidate();
				}
				ret->left = nullptr;
				ret->right = nullptr;
				ret->child = nullptr;
				n--;
			}
			return ret;
		}

		void decrease_value(fibo_heap_node<T>* node, const T& new_value) {
			if (node->value < new_value) {
				throw std::exception("new value should smaller than old value");
			}
			node->value = new_value;
			auto parent = node->parent;
			// 父节点大于当前节点，需要断开、递归向上断开
			if (parent && parent->value > node->value) {
				cut(node, parent);
				cascade_cut(parent);
			}
			if (node->value < min_root->value) {
				min_root = node;
			}
		}

		void delete_node(fibo_heap_node<T>* node) {
			decrease_value(node, min_root->value);
			extract_minimun();
		}

		// 实用打印函数
		std::vector<fibo_heap_node<T>*> get_root_list() {
			std::vector<fibo_heap_node<T>*> ret;
			ret.push_back(min_root);
			for (auto root = min_root->right
				; root != min_root
				; root = root->right) {
				ret.push_back(root);
			}
			return ret;
		}

		// 实用打印函数
		std::vector<fibo_heap_node<T>*> get_child_list(fibo_heap_node<T>* parent) {
			std::vector<fibo_heap_node<T>*> ret;
			if (!parent->child)
				return ret;
			ret.push_back(parent->child);
			for (auto child = parent->child->right
				; child != parent->child
				; child = child->right) {
				ret.push_back(child);
			}
			return ret;
		}

		friend std::ostream& operator<<(std::ostream& o, fibo_heap<T> heap) {
			o << "<<<<<print fibo heap>>>>>" << std::endl << "nums: " << heap.n << std::endl;
			auto root_list = heap.get_root_list();
			std::queue<fibo_heap_node<T>*> node_queue;
			o << "root list: ";
			for (auto& t : root_list) {
				o << t->value << " ";
				auto childs = heap.get_child_list(t);
				for (auto& t : childs) {
					node_queue.push(t);
				}
			}
			o << std::endl;
			while (!node_queue.empty()) {
				o << " ===== " << std::endl;
				for (size_t nums = node_queue.size(); nums != 0; nums--) {
					auto node = node_queue.front();
					node_queue.pop();
					o << "node: " << node->value << " parent is: " 
						<< node->parent->value << std::endl;
					auto childs = heap.get_child_list(node);
					for (auto& t : childs) {
						node_queue.push(t);
					}
				}
			}
			o << "<<<<<print over>>>>>" << std::endl << std::endl;
			return o;
		}
	};

}

int main(){
	using namespace lyc_algorithm;
	fibo_heap<int> fibo_heap_test;
	fibo_heap_test.insert(1);
	fibo_heap_test.insert(2);
	fibo_heap_test.insert(-2);
	fibo_heap_test.insert(-4);
	fibo_heap_test.insert(0);
	fibo_heap_test.insert(12);
	auto mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	fibo_heap<int> other;
	other.insert(233);
	other.insert(666);
	other.insert(255);
	other.insert(-1234);
	fibo_heap_test.merge(other);
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	mini = fibo_heap_test.extract_minimun();
	std::cout << "extract minimum: " << mini->value << std::endl;
	delete mini;
	auto node = new fibo_heap_node<int>(1);
	fibo_heap_test.insert(node);
	fibo_heap_test.insert(0);
	fibo_heap_test.insert(-7);
	fibo_heap_test.insert(6);
	fibo_heap_test.insert(-2);
	fibo_heap_test.insert(23);
	std::cout << fibo_heap_test.extract_minimun()->value << std::endl;
	fibo_heap_test.decrease_value(node, -1);
	std::cout << fibo_heap_test.get_minimum() << std::endl;
	return 0;
}
```
# 后记
- 如果你是按顺序阅读的，那么读完树这一章，你应该会有一个疑问，为什么这部分内容算在了算法里面。是的，虽然这部分内容看起来更像是数据结构，但是其实这些数据结构的本质思想，已经不再是简简单单的链表、队列、动态表这些，而是更高级的思想。比如如何利用二分法、均摊、附加标记等方式或思路，大幅度提高某些场景下的运行效率。
- 更幸运的是，大部分情况下，我们可以在很多语言中直接找到基于这些数据结构的实用工具。尤其是红黑树、B树，他们俩几乎构成了常用平衡搜索树的绝大部分江山。是我们进入$O(logn)$的最佳帮手。
# 在写作本章节时记录的博客问题
- 在使用mathjax的\\$\\$对儿中，如果直接写\[x\]，则会显示出来一个[x]，如果想要正常显示\[x\]，需要写为\\\\[x\\\\]。
    - markdown的转义。。。有的时候真的令人无语。