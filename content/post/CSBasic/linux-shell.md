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
- 三剑客之一：主要使用方式是预处理+逐行处理+最终处理，常用于格式化输出、统计等工作
- 基本结构
    ```bash
    awk [各命令行参数] 'BEGIN{} {/pattern1/ action1;action2;} \
                         {/pattern2/ action3;action4;} ... END{}' {filenames}
    ```
    - awk通常逐行读入并根据各块内斜杠/对儿内的pattern进行匹配，每当匹配成功，执行对应块内的action，不同pattern之间互不影响
    - BEGIN块最先执行、END块最后执行，其他各块之间按顺序依次执行
    - action内容和脚本语言类似，有一套自己的语法
    - 注意使用**单引号**'，在单引号对儿内部，仍然可以使用双引号对儿""代表内部是字符串
- pattern语法（正则表达式[Wiki](https://en.wikipedia.org/wiki/Regular_expression)）
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
    | [:lower:] | 任意小写字母，参考[POSIX字符组](https://blog.csdn.net/shangboerds/article/details/7555332) |
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
- 三剑客之二：处理、编辑文本文件，和awk相比，更擅长修改文件。默认输出结果到控制台。
    - sed是面向行的处理，读入每一行，保存到缓存区（模式空间），匹配并做相应动作，不会修改源文件。
    - 所使用的正则表达式和awk的正则语法一致，也用斜杠对儿/pattern/作为表达式范围标志。
- 基本语法
    ```bash
    # 使用脚本，或脚本文件，处理输入文件
    sed [-e <script>] [-f <scriptFiles>] [inputFiles]
    ```
- 动作
    | 命令 | 功能 |
    | --- | --- |
    | a | 追加 |
    | c | 更改 |
    | i | 插入 |
    | d | 删除 |
    | s | 替换 |
    | p | 打印 |
    | = | 用于打印被匹配行的行号 |
    | n | 跳到下一行 |
    | r | 将内容读入文件 |
    | w | 将匹配内容写入文件 |
    | y | 一对一替换 |
    | q | 退出 |
- 脚本结构：
    1. 不指定行的操作：触发条件(匹配，行) + 动作 + 动作参数 + 动作 + 参数...例如：
        ```bash
        # -e 可省略
        # 匹配123的行并增加hello
        sed '/123/ahello' test.txt
        # 最后一行添加hello
        sed '$ahello' test.txt
        # 替换语法s/x/y/，每行只替换第一个
        sed 's/from/to/' test.txt
        # 替换，行内出现多次也替换
        sed 's/from/to/g' test.txt
        # 替换，行内替换第2个匹配位置
        sed 's/from/to/2' test.txt
        # 替换并写入新文件
        sed 's/from/to/gpw output.txt' test.txt
        # 从input.txt读入内容到test.txt的第3行后
        sed '3r input.txt' test.txt
        # 删除每行的最后两个字符
        sed 's/..$//g' test.txt
        # a->A, b->B, c->C
        sed 'y/abc/ABC/' test.txt
        ```
    1. 指定行的操作：运算符, ~ +
        ```bash
        # 在第3行后添加新行2333
        sed '3a2333' test.txt
        # 第3行插入hello
        sed '3ihello' test.txt
        # 第一行替换为hello
        sed '1chello' test.txt
        # 删掉奇数行（每两个删一个）
        sed '1~2d' test.txt
        # 删掉1到2行
        sed '1,2d' test.txt
        # 打印匹配#的行及其后1行
        sed '/#/,+1p' test.txt
        ```
    1. 复合动作：花括号{}
        ```bash
        # 删掉1到3行中匹配123的行
        sed '1,3{/123/d}' test.txt
        # 打印最后一行的行号和内容
        sed '${=;p}' test.txt
        # 匹配hello，并删除其下一行的内容
        sed '/hello/{n;d}' test.txt
        ```
    1. 取反：运算符!
        ```bash
        # 删掉除了1,2行的所有行
        sed '1,2!d' test.txt
        ```
    1. 结合
        ```bash
        # 匹配没有#号的行并替换@为#
        sed '/#/!s/@/#/g' test.txt
        ```
    1. 多重编辑：注意前后顺序有影响
        ```bash
        # 先删掉1至3行，再进行替换
        sed -e '1,3d' -e 's/Rex/Lyc/g' test.txt
        ```
- 参考：[shell脚本-sed的用法](https://blog.csdn.net/wdz306ling/article/details/80087889)、[sed详解](https://blog.csdn.net/w757052816/article/details/119874525)
### grep
- 三剑客之三：强大的文本搜索工具，家族成员包括grep、egrep、fgrep，后两者只是前者的一种扩展。
    - 匹配指定文件中的各行，默认打印匹配成功的行
    - 默认使用基础正则表达式
- 基本语法
    ```bash
    # 输入匹配用的正则表达式，以及待匹配的输入文件
    grep expression files
    ```
- 常用参数
    | 参数 | 意义 |
    | --- | --- |
    | -a | 对二进制文件依然尝试匹配 |
    | -c | 统计匹配成功的行数总和 |
    | -A n | 显示匹配行和之后的n行 |
    | -B n | 显示匹配行和之前的n行 |
    | -C n | 显示匹配行前后各n行 |
    | -e expression | 指定使用的匹配字符串 |
    | -E | 扩展正则模式(和基础正则相比，转义的含义恰好相反) |
    | -f files | 指定基础正则表达式所在文件，一行一个 |
    | -F | 将表达式视为普通字符串，进行朴素模式匹配 |
    | -H | 在打印匹配行前，额外显示匹配行所在文件 |
    | -i | 忽略大小写 |
    | -l（小写的L） | 不打印匹配行，只列出匹配成功的文件名 |
    | -L | 列出无法匹配的文件名 |
    | -n | 列出行号 |
    | -d read/skip/recurse | 输入文件含有目录时，处理该目录/跳过/递归处理该目录 |
- 用例
    ```bash
    # 在test.txt中查找hello
    grep hello test.txt
    # 使用基础正则查找hellohello
    grep "\(hello\)\{2\}" test.txt
    # 使用扩展正则
    grep -E "(hello){2}" test.txt
    # 递归查找当前目录下文件，扩展正则匹配
    grep -d recurse -E "(hello){2}" ./
    ```
- 参考：[grep 命令](https://www.runoob.com/linux/linux-comm-grep.html)
### find & locate：
- find：相对较慢的通用查询方式，但可以指定不同属性查找文件，属性包括：名称、文件类型、文件大小、文件归属、文件权限、文件时间等等
    ```bash
    # 基本语法
    # 指定路径在前（默认从当前位置开始），指定搜索内容在后
    find path expression -option [-print] [-exec -ok command {} \;]
    # 查找指定名称的目录
    find ./ -type d -name Projects
    # 使用通配符
    find ./ -name "*.log"
    # 查找指定777权限
    find ./ -perm 777
    # 查找空目录
    find ./ -type d -empty
    # 查找空文件
    find ./ -type f -empty
    # 查找用户为root的文件
    find / -user root
    # 查找属组是root的文件
    find / -group root
    # 查找大小在30M到100M的文件
    find ./ -size +30M -size -100M -type f
    # 查找777权限，打印并改为700，结尾{} \;是必需的
    find ./ -perm 777 -print -exec {} \;
    ```
- locate：相对较快的查询，但需要提前用updatedb构建数据库，但只能查询名字，一般需要单独安装
## 其他常用指令
1. 二进制查看hexdump
    | 参数 | 含义 |
    | --- | --- |
    | -n length | 显示前length个字节 |
    | -C | 单字节十六进制和ascii |
    | -c | ascii |
    | -d | 双字节十进制 |
    | -x | 双字节十六进制 |
    | -s pos | 偏移pos个字节开始输出 |
    | -e '"format" a/b "format1 "format2"' | 以指定输出格式打印[具体参考](https://blog.csdn.net/zsj1126/article/details/105770068/) |
1. 压缩tar：
    | 参数 | 含义 |
    | --- | --- |
    | -c | 创建新的tar包 |
    | -x | 解tar包 |
    | -t | 列出tar包内容列表 |
    | -r | 附加新的文件到tar |
    | -v -vv | 打包、解包时显示文件名、显示文件全部属性 |
    | -k | 保留旧文件不覆盖 |
    | -z -Z -j | 调用gzip、compress、bzip2进行解压缩 |
    | -f file | 从file解包，或打包成file |
    | -C path | 解包到指定路径 |
    - 示例：
        ```bash
        # 将当前目录下所有文件打入package包
        tar -czvf package.tar.gz ./*
        # 解压到/tmp
        tar -zxvf package.tar.gz -C /tmp/
        ```
1. 用户管理useradd、groupadd、userdel、groupdel、usermod、groupmod、groups、id
    - 这些命令主要对/etc/passwd、/etc/shadow、/etc/group三个文件进行维护，[字段含义参考](http://www.javashuo.com/article/p-mwuizuri-pq.html)
    ```bash
    # 创建用户liyicheng，并设定：登陆位置/tmp，主属组lyc
    # 附加属组ubuntu，home下不创建用户目录（-M），不创建同名用户组（-N）
    useradd -d /tmp/ -g lyc -G ubuntu -M -N liyicheng
    # 修改用户liyicheng：主属组为ubuntu，增加（-a）附加属组mysql
    # 改名为liyicheng2，改登录shell为zsh
    usermod liyicheng -g ubuntu -G mysql -a -l liyicheng2 -s /usr/bin/zsh
    # 删除用户在passwd、group、shadow、gshadow中的内容，并删除其主目录下文件
    userdel -r liyicheng
    # 查看属组
    groups liyicheng
    # 查看uid、gid等
    id liyicheng
    # 添加普通用户组（添加-r则为系统用户组）
    groupadd li
    # 删除用户组（此时要求没有以li为主用户组的用户）
    groupdel li
    # 修改用户组li的gid为2333，名字为newli
    # groupmod 改组名和gid都容易会引起混乱，慎用
    groupmod -g 2333 -n newli li
    ```
1. 权限管理chmod、chown
    ```bash
    # 递归修改当前路径以下的文件、符号链接的用户为liyicheng，属组为li
    # 注：-L 不修改所有的符号链接（软链接），-H不修改目录的符号链接
    chown -R liyicheng:li ./*
    # -c打印修改、-f屏蔽错误信息、-v为verbose、-R递归处理
    # ugoa分别是用户、同组、其他、所有人
    # rwx读写运行，Xst
    # 注：chmod永远不会修改符号链接的权限
    chmod [-cfvR] [ugoa...][[+-=][rwxXst]...][,...] file
    ```
1. 系统状态
    - top、htop：查看系统综合信息和各进程
    - free：查看内存信息
    - df：查看磁盘使用情况
    - du：查看文件的大小（默认只显示目录的占用大小）
        | 参数 | 含义 |
        | --- | --- |
        | -a | 显示所有文件（含目录） |
        | -b | 显示文件实际大小，单位byte |
        | -c | 统计大小之和 |
        | -d n | 只显示到第n层（n层以下只统计，不详细列出） |
        | --inodes | 显示inode信息 |
        | -s | 只列出整体结果 |
        | -X file | 去除指定文件不计入统计 |
        | --exclude=pattern | 去除匹配不计入统计（使用通配符?和*，非正则表达式） |
1. 进程管理ps、kill
1. 文件查看cat、more、less、tail、head、sort
1. 链接ln
1. 网络工具集合
    - 嗅探nmap
        ```bash
        # 嗅探指定端口
        nmap ip -p port
        # 嗅探全部端口
        nmap ip
        ```
    - ssh详见ssh章节
    - 网络传输scp
    - 网络状态netstat
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
- 拷贝、转换文件dd：
```bash
# 从输入拷贝到输出
dd if=input.txt of=output.txt
# 拷贝512字节的0
dd if=/dev/zero of=zero.bin count=1
# 交换每两个相邻字节
dd if=input.txt of=output.txt conv=swab
```
## 一些建议
1. 对于rm，可以替换为mv到临时文件夹，并定期清理，尽量避免使用rm，尤其禁止使用rm -rf