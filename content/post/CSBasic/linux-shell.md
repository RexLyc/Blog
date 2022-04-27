---
title: "边学边用linux-命令行篇"
date: 2022-02-10T14:54:43+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 命令行
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
math: true
---
需要熟练掌握命令行是Linux系列系统最大的特点之一。本文从实用角度出发编写。
<!--more-->
## 核心指令
![键盘图](/images/Linux/vi-vim-cheat-sheet-sch.gif)
### vi/vim
- 关系：vim是vi的继承和发展，下面以vim为主，和vi不兼容处会**进行标注**
- Command（命令）模式
        <table>
            <tr>
                <th>目标</th>
                <th>按键</th>
                <th>说明</th>
            </tr>
            <tr>
                <th rowspan="6"> 切换模式 </th>  <!-- 使用th会添加横线 -->
                <td> iao/IAO </td>
                <td> 于光标前后、行前后、行上下切到输入模式 </td>
            </tr>
            <tr>
                <td> R </td>
                <td> 替换模式 </td>
            </tr>
            <tr>
                <td> : </td>
                <td> 切到最后行模式 </td>
            </tr>
            <tr>
                <td> 小v </td>
                <td> 可视模式 </td>
            </tr>
            <tr>
                <td> 大V </td>
                <td> 可视行模式（一行全被选中） </td>
            </tr>
            <tr>
                <td> Ctrl+v/V </td>
                <td> 可视块模式（选中的是一个跨行的矩形） </td>
            </tr>
            <tr>
                <th rowspan="19"> 快速移动 </th>
                <td> w/W </td>
                <td> 向后一词 </td>
            </tr>
            <tr>
                <td> b/B </td>
                <td> 向前一词 </td>
            </tr>
            <tr>
                <td> {/} </td>
                <td> 段落首尾 </td>
            </tr>
            <tr>
                <td> hjkl </td>
                <td> 左下上右 </td>
            </tr>
            <tr>
                <td> Ctrl+f/d </td>
                <td> 向下一页半页 </td>
            </tr>
            <tr>
                <td> Ctrl+b/u </td>
                <td> 向上移动一页半页 </td>
            </tr>
            <tr>
                <td> Ctrl+e/y </td>
                <td> 上滚下滚（光标仍在原行） </td>
            </tr>
            <tr>
                <td> ^ 0 | </td>
                <td> 移动到行首 </td>
            </tr>
            <tr>
                <td> $ </td>
                <td> 行尾 </td>
            </tr>
            <tr>
                <td> e/E </td>
                <td> 词尾 </td>
            </tr>
            <tr>
                <td> f/F </td>
                <td> 当前行向后向前查询字符（字符由下一个输入给出） </td>
            </tr>
            <tr>
                <td> H </td>
                <td> 移动光标至屏幕顶行 </td>
            </tr>
            <tr>
                <td> L </td>
                <td> 移动光标至屏幕底行 </td>
            </tr>
            <tr>
                <td> M </td>
                <td> 移动光标至屏幕中间行 </td>
            </tr>
            <tr>
                <td> 数字+空格 </td>
                <td> 向后移动若干字符 </td>
            </tr>
            <tr>
                <td> G </td>
                <td> 最后一行 </td>
            </tr>
            <tr>
                <td> gg </td>
                <td> 最初行 </td>
            </tr>
            <tr>
                <td> nG </td>
                <td> 移动到第n行 </td>
            </tr>
            <tr>
                <td> n+回车 </td>
                <td> 向下移动n行 </td>
            </tr>
            <tr>
                <th rowspan="5"> 修改 </th>
                <td> x/X </td>
                <td> 向后/前删除 </td>
            </tr>
            <tr>
                <td> r </td>
                <td> 替换字符（光标处替换为下一个输入字符） </td>
            </tr>
            <tr>
                <td> p/P </td>
                <td> 向后/前粘贴 </td>
            </tr>
            <tr>
                <td> s/S </td>
                <td> 删除当前字符/行并切到输入模式、 </td>
            </tr>
            <tr>
                <td> J </td>
                <td> 合并两行 </td>
            </tr>
            <tr>
                <th rowspan="2"> 复制 </th>
                <td> y拷贝 </td>
                <td> 拷贝（yy当前行、nyy复制n行、yG复制当前行到末行、y0复制当前字符到行首、y$复制当前字符到行尾） </td>
            </tr>
            <tr>
                <td> Y </td>
                <td> 整行拷贝 </td>
            </tr>
            <tr>
                <th rowspan="3"> 搜索 </th>
                <td> /word </td>
                <td> 向后搜索 </td>
            </tr>
            <tr>
                <td> ?word </td>
                <td> 向前搜索 </td>
            </tr>
            <tr>
                <td> n/N </td>
                <td> 下/上一个搜索结果 </td>
            </tr>
            <tr>
                <th rowspan="5"> 动作变更 </th>
                <td> u </td>
                <td> 撤销 </td>
            </tr>
            <tr>
                <td> U </td>
                <td> 行内撤销 </td>
            </tr>
            <tr>
                <td> Ctrl+r </td>
                <td> 重做（u的反向） </td>
            </tr>
            <tr>
                <td> . </td>
                <td> 重复上一动作 </td>
            </tr>
            <tr>
                <td> 数字+命令 </td>
                <td> 重复执行命令 </td>
            </tr>
        </table>
                
- Insert（输入）模式
    - 在此模式下可以像图形化文本编辑器一样，向文件输入字符
- Last Line（最后行）模式
    - 快速移动：数字（移动到该行）
    - :wq、q、q!：保存退出、退出、不保存退出
        - :wq 等价于 ZZ（命令模式下）
        - :wq 等价于 ZQ（命令模式下）
    - :w [filename] 写入指定文件
    - :r [filename] 插入指定文件的内容到当前文件
    - :! [command] 暂时返回shell，执行command
    - :n1,n2s/word1/world2/g 替换n1行到n2行之间的word1为word2
        - 其中n2可用$符号代表最后一行，1,$也直接等价于%
- 实用配置：
    - 显示行号  :set nu
    - 取消行号显示  :set nonu
- 名词解释
    - 段落：一系列非空行是一个段落
    - 句子：以. ! ?结尾的，且后继字符为空格、水平制表符、换行符的一系列单词
    - 单词：默认情况下，是由字母、数字、下划线和其他部分非空字符（如~）组成的连续字符串，其他任何字符都视作分割
- 参考：[菜鸟教程](https://www.runoob.com/linux/linux-vim.html)、[将vim配置成强大的IDE编辑工具](https://blog.csdn.net/qq_26708669/article/details/121057164)
### awk
- 三剑客之一
- 基本结构
    ```bash
    awk [各命令行参数] 'BEGIN{} {/pattern1/ action1;action2;} \
                         {/pattern2/ action3;action4;} ... END{}' {filenames}
    ```
    - awk通常逐行读入并根据各块内斜杠/对儿内的pattern进行匹配，每当匹配成功，执行对应块内的action，不同pattern之间互不影响
    - BEGIN块最先执行、END块最后执行，其他各块之间按顺序依次执行
    - action内容和脚本语言类似，有一套自己的语法
    - 注意使用**单引号**'，在单引号对儿内部，仍然可以使用双引号对儿""代表内部是字符串
- pattern语法（正则表达式）
    | 符号 | 描述 |
    | --- | --- |
    | ^$ | 行首尾定位符 |
    | . | 匹配1个任意单个字符 |
    | * | 匹配$[0,+\infty]$个前导字符 |
    | + | 匹配$[1,+\infty]$个前导字符 |
    | ? | 匹配$[0,1]$个前导字符 |
    | [] | 匹配一次括号内某个字符 |
    | [^ ] | 匹配一次括号内字符集以外的任意字符 |
    | () | 匹配括号内的字符串整体 |
    | \| | 或，用于组合多种匹配情况 |
    | \\ | 转义 |
    | x{m[,[,n]]} | 匹配$m$、$[m,+\infty]$、$[m,n]$次（需awk版本支持） |
    > 注意只要正则表达式能成立（能匹配到该行的部分或全部字符），则会执行后续动作，即使匹配空。这点在使用*且不指定边界时应当尤其注意。
- action语法
    - 运算符：
    | 符号 | 描述 |
    | --- | --- |
    | = += -= *= /= %= ^= **= | 赋值 |
    | \|\| && | 逻辑运算 |
    | ~ !~ | 正则匹配、正则失配（用法见例子） |
    | < <= > >= != == | 关系运算 |
    | + - * / & !| 加减乘除与非|
    | ^ *** | 求幂 |
    | ++ -- | 前后缀自增自减 |
    | $value | 引用变量value |
    | \<space\> | 字符串链接 |
    | ?: | 三目运算符 |
    | in | 数组中是否存在 |
    - 内置变量：
    | 名称 | 用处 |
    | --- | --- |
    | $0 | 当前行（此次处理的输入） |
    | \\$1~\\$n | 当前记录正则匹配的第i段 |
    | FS | 字段分隔符 |
    | RS | 行分隔符 |
    | NF | 行内字段数 |
    | NR | 行数 |
    | OFS | 输出用的字段分隔符 |
    | ORS | 输出用的行分隔符 |
    > 注意只有$1等需要前置，其他变量不需要，就像是C、python中的变量一样使用即可。
    - 控制语句
        1. 控制语句复杂时，建议单独创建文件，分多行书写，行尾不需要写;
        1. if、else、圆括号、花括号等同于C语言
        1. 支持for(;;)循环体、支持break、continue
        1. 支持for( a in b)循环
        1. 定义变量不需要写类型
        1. awk中的数组本质上都是map，可以使用字符串等做索引
    - 函数定义
        ```
        function name(arg1,arg2,...) {
            ....
            return ...
        }
        ```
- 命令行参数：
    - -F fs 或 --field-separator fs ：给出后续输入文本的字段分隔符
    - -f scriptfile 或 --file scriptfile ：从指定文件中加载awk命令
    - -v var=value 或 --asign var=value ：定义变量
- 参考：[Linux三剑客之awk](https://www.cnblogs.com/ginvip/p/6352157.html)、[awk内置函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html)
### sed
- 三剑客之二
### grep
- 三剑客之三
### find & locate：
## 其他常用指令
1. 网络工具集合
    ```bash
    # 嗅探指定端口
    nmap ip -p port
    # 嗅探全部端口
    nmap ip
    ```
1. 终端复用tmux
    1. 基本用法
        ```bash
        # 创建名为test的tmux会话
        tmux new -s test
        # 进入名为test的tmux会话
        tmux attach -t test
        # 进入第一个会话（如果你只有一个的话这样更快）
        tmux a
        # 杀死名为test的会话
        tmux kill -t test
        # 杀死未处于使用状态（未attach）的所有会话
        tmux kill-session
        ```
    1. tmux内
## Shell编程
1. 基本概念
    1. 标准输入输出
    1. 重定向
    1. 管道
    1. 前后台
    1. 系统变量
1. 核心语法
1. 经典例子
1. Shell环境
    - .bashrc
    - .bash_profile
1. 启动环境
    - init.d
    - rcX.d
## 其他指令收集
- 查看系统发行版：
```bash
# Red Hat系 & Ubuntu系
cat /etc/os-release
# Ubuntu系
cat /etc/issue
```