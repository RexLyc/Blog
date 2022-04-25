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
---
需要熟练掌握命令行是Linux系列系统最大的特点之一。本文从实用角度出发编写。
<!--more-->
## 核心指令
![键盘图](/images/Linux/vi-vim-cheat-sheet-sch.gif)
1. vi/vim
    - 关系：vim是vi的继承和发展，下面以vim为主，和vi不兼容处会**进行标注**
    - Command（命令）模式
        - 切模式：iao/IAO切到输入模式、R替换模式（持续替换）、:切到最后行模式、v可视模式、V可视行模式（一行全被选中）、Ctrl+v/V可视块模式（选中的是一个跨行的矩形）
            - 可视模式可以仍可以快速移动命令
        - 快速移动：w/W向后一词、b/B向前一词、{/}段落首尾、hjkl左下上右、Ctrl+f/d向下一页半页、Ctrl+b/u向上移动一页半夜、Ctrl+e/y上滚下滚（上下滚光标位置不变）、^0|/$行首尾（行首最常用的还是0）、e/E词尾、f/F当前行向后向前查询字符（字符由下一个输入给出）、H移动光标至屏幕顶行、L移动光标至屏幕底行、M移动光标至屏幕中间行、数字+空格（向后移动若干字符）、G最后一行、gg最初行、nG移动到第n行、n+回车（向下移动n行）
        - 修改：x/X向后/前删除、r替换字符（光标处替换为下一个输入字符）、p向后粘贴、P向前粘贴、s删除当前字符并切到输入模式、S删除当前行并切到输入模式、J合并两行
        - 复制：y拷贝（yy当前行、nyy复制n行、yG复制当前行到末行、y0复制当前字符到行首、y$复制当前字符到行尾）、Y整行拷贝
        - 搜索替换：/word向后搜索、?word向前搜索、n/N下/上一个搜索结果
        - 动作变更：u撤销、U行内撤销、Ctrl+r重做（u的反方向）
        - 重复：.重复上一动作
        - 数字+大部分命令：重复执行若干次
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
    - 实用技巧记录：
        - 
    - 实用配置：
        - 显示行号  :set nu
        - 取消行号显示  :set nonu
    - 名词解释
        - 段落：一系列非空行是一个段落
        - 句子：以. ! ?结尾的，且后继字符为空格、水平制表符、换行符的一系列单词
        - 单词：默认情况下，是由字母、数字、下划线和其他部分非空字符（如~）组成的连续字符串，其他任何字符都视作分割
    - 参考：[菜鸟教程](https://www.runoob.com/linux/linux-vim.html)、[将vim配置成强大的IDE编辑工具](https://blog.csdn.net/qq_26708669/article/details/121057164)
1. awk
1. sed
1. grep
1. find
## 其他常用指令
1. 网络工具
    ```bash
    # 嗅探指定端口
    nmap ip -p port
    # 嗅探全部端口
    nmap ip
    ```
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