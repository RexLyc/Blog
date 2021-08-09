---
title: "算法导论其一：分析（施工中）"
date: 2021-07-08T22:41:56+08:00
categories:
- 计算机科学与技术
- 算法
tags:
- 算法系列
keywords:
- tech
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/algorithm.jpg
math: true
---
工欲善其事必先利其器，写出优良算法的基础是拥有对算法性能衡量的能力。一起回顾一下时间复杂度的计算吧。
<!--more-->
> 假设计算机无限快，并且计算机存储器是免费的，那么你还有任何理由来研究算法吗？
> <p align="right">——《算法导论》</p>
# 关注点
0. 性能：尽量跑的快一点
1. 安全：比如不要泄露内存
2. 扩展：泛用性很强（对字符串排序、对数字排序）
3. 易用性：接口简单
4. 鲁棒性：稳定（不要忽快忽慢，忽然崩溃）
# 常用性能衡量工具
0. 数学符号（渐进记号）
    - 定义域为自然数域 $\mathrm{N}=\\{0,1,2,...\\}$
    - 目的：衡量并比较算法性能优劣
    - $\Theta$记号：$\Theta(g(n))=\\{f(n);\exists c_1,c_2,n_0,\forall n\ge n_0,0\le c_1 g(n) \le f(n) \le c_2 g(n)\\}$
        - 人话：一定输入规模之后，$f$可以被$g$夹着。渐近地代表了一个函数的上界和下界。
    - $\mathrm{O}$记号：$\mathrm{O}(g(n))=\\{f(n);\exists c,n_0,\forall n\ge n_0,0\le f(n) \le c g(n)\\}$
        - 人话：一定输入规模之后，$f$不会比$g$大。$\Theta$记号是隐含着$\mathrm{O}$的。
    - $\Omega$记号：$\Omega(g(n))=\\{f(n);\exists c,n_0,\forall n\ge n_0,0\le c g(n)\le f(n) \\}$
        - 人话：一定输入规模之后，$f$不会比$g$小。$\Theta$记号也是隐含着$\mathrm{O}$的。
    - $\mathrm{o}$记号：$\mathrm{o}(g(n))=\\{f(n);\forall c, \exists n_0,\forall n\ge n_0,0\le f(n) \le c g(n)\\}$
        - 人话：一定是非渐近紧确的上界，标志着在输入规模足够之后，$f$是$g$的等价无穷小了。
    - $\omega$记号：$\omega(g(n))=\\{f(n);\forall c, \exists n_0,\forall n\ge n_0,0\le c g(n)\le f(n) \\}$
        - 人话：一定是非渐近紧确的下届，标志着在输入规模足够之后，$g$是$f$的等价无穷小了。
    - 注意：
        1. $\Theta \mathrm{O} \Omega$记号都是不保证渐近紧确的。
        2. 传递性：五种符号都支持
        3. 自反性：$\Theta \mathrm{O} \Omega$
        4. 对称性：$\Theta$
        5. 转置对称性（即$fg$对换位置）：$\mathrm{O}$和$\Omega$，$\mathrm{o}$和$\omega$
    - 运算约定
        1. 最常用的写法是：$f(n)=\Theta(g(n))$，更标准的写法是$f(n)\in \Theta(g(n))$。
        2. 出现在一个“等式”中，如：$2n^2+3n+1=2n^2+\Theta(n)$，此时渐近符号代表某个不在乎其名字的匿名函数。
        3. 出现在“等式”两边，如：$2n^2+\Theta(n)=\Theta(n^2)$，此时代表，无论左侧的匿名函数如何选择，总有一个右边的匿名函数能使等式成立。**可以整体理解为，任意左侧的属于右侧**
    - 常用数学：
        1. 任何底大于1的指数函数都比任何多项式函数增长的更快：$n^b=\mathrm{o}(a^n),a>1$
        2. 任何正多项式函数都比多相对数函数增长的更快：$lg^bn=\mathrm{o}(n^a),a>0$
        3. 易知：$n!=\mathrm{o}(n^n),n!=\omega(2^n)$，并可用斯特林公式证明：$lg(n!)=\Theta(nlgn)$
1. 主方法（Master Method）
    1. 形式：对递归表达式$T(n)=aT(\frac{n}{b})+f(n)$，要求$f(n)$渐近趋正（n够大的时候都大于0），$b>1,a\ge 1$
    2. 比较：$f(n)$ vs $n^{log_b^a}$
        - 若存在常数$\varepsilon>0$，使得$f(n)=\mathrm{O}(n^{(log_b a)-\varepsilon})$，则有$T(n)=\Theta(n^{log_b a})$
        - 若$f(n)=\Theta(n^{log_b a})$，则$T(n)=\Theta(n^{log_b a} lgn)$
        - 若存在常数$\varepsilon>0$，使得$f(n)=\Omega(n^{(log_b a)+\epsilon})$，且对常数$c<1$与所有足够大的$n$，有$af(\frac{n}{b})\le cf(n)$，则$T(n)=\Theta(f(n))$
    3. 主方法并不能适用于所有递归表达式，有些式子无法满足所有
    4. 主方法是有严格证明的，可以看，但没必要（折磨自己）。
2. 画递归树
    - 通常可以先用递归树获得一个不太严谨的答案，然后使用代换法证明
    - 核心思想是统计出出所有的叶节点数量，并计算每一层的任务量
3. 代换法（猜想并归纳证明）
    0. 例如：$T(n)=4T(n/2)+n$
    1. 猜想是$T(n)=\mathrm{O}(n^2)$（不紧确）
    2. 设$T(k) \le c_1 k^2 - c_2 k$
    3. 代换原始递归式并证明可以满足猜想
    4. 证明边界情况满足（例如T(1)或T(2)等）
    4. 需要较高的猜想技巧，另外该方法并不保证猜出渐近紧确的界
# 平摊分析（等待补充）
部分算法、数据结构，它的单一操作可能最坏、但平摊耗费较小。不保证实时性能。对于这类算法和数据结构，不能用常规方法分析，需要对其平均性能进行估计。

0. 例子
1. 聚集分析
2. 记账法
3. 势能法
# 竞争分析（等待补充）
0. 分类
1. 应用
2. 方法
3. 举例
# 参考资料
*学点英语，主要是尽量把术语发音正确噢* :winking_face_with_tongue:<a href="https://www.bilibili.com/video/BV1Tb411M7FA?from=search&seid=3716615071312119347" target="_blank">MIT6.046Jのbilibili传送门</a>
# 本节博客搭建过程中的坑
0. 添加内嵌html支持，需要修改toml，令markdown引擎支持内嵌html（unsafe模式）
1. <a href="https://note.qidong.name/2018/03/hugo-mathjax/" target="_blank">使用MathJax在Hugo的Markdown中绘制公式</a>
    - 第一步：添加一个单独的html于主题文件夹partials下，并引入所需的js库，css内容。
    - 第二步：去head.html中利用插值表达式引用该html。
    - 第三步：最好是在markdown开头的Front Matter部分添加math: true。也可以在toml中全局启用MathJax。
2. 使用bootcdn对js、css等进行加速，注意只能使用主题中限定的版本。