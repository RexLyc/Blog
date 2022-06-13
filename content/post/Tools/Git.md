---
title: "Git"
date: 2021-08-11T00:00:13+08:00
categories:
- 实用工具
tags:
- 实用工具
- 施工中
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/git.png
---
据说git是大佬Linus用2周时间做出来的小工具。工具虽小，却成为了当今开源世界的重要基石。本文计划一站式解决Git从原理到实践上的核心问题。本文尚未完成
<!--more-->
## .git/目录结构
- 子目录
    - hooks/：其内部有一系列钩子函数示例，以git定义的脚本方式，可以进行git操作各种阶段的自动化工作
    - info/：内部存储exclude信息，即.gitignore对应的信息
    - logs/：存储所有更新的记录，HEAD文件保存本地的所有操作记录，refs文件夹内有stash（暂存）、heads文件夹（存储本地每个分支的操作记录）、remotes文件夹（远程每个分支的操作记录）
    - modules/：用于存储git submodule。内部又是若干个独立的.git目录，当前项目有多少git子模块，就有多少个独立的.git目录
    - objects/：存放所有的git对象，哈希值（SHA-1哈希算法）共40位，前两位做文件夹名称，后38位为文件名
    - refs/：内部有多层目录，很好理解，都是用来存储特殊的git对象哈希值，指向每个分支当前的位置
- 文件
    - config：仓库配置，远程url等
    - COMMIT_EDITMSG：最新一次提交的commit message
    - description：仓库描述
    - index：就是git的暂存区
    - FETCH_HEAD：指向从远程仓库fetch的分支的最后位置
    - ORIG_HEAD：HEAD指针的前一个状态（可用于git checkout -等操作）
    - HEAD：当前正使用的分支引用名称
    - packed-refs：当ref文件过多时出现，用于优化
## 核心原理
<img src="/images/tools/git-object-model.png"></img>
<center>图片引用自Git Book</center>

1. git基本原理：内容寻址文件系统，即git是依靠存储内容本身来进行存储和访问的。每当用户插入部分内容，将会获得一个以文件内容为输入的哈希值作为键值，此后可以通过该键值访问该部分内容。存储的文件对象是完整的，而不是变动的增量。
    - git hash-object -w：从输入获取哈希值，并存储文件对象到git仓库
    - git cat-file：查看指定的哈希值所存储的文件
1. 对象类型：将文件存储，文件夹组织起来
    - blob：只存储内容而没有文件名等文件系统系统信息
    - tree：存储一个完整的文件，文件夹（递归）
        - git update-index：将指定哈希值（代表文件内容）、指定文件名、指定文件类型的文件加入暂存区。当然也有可能是一个指定树对象哈希名、指定文件夹名称、指定文件夹类型加入缓存区。
        - git write-tree：将暂存区内容写入到一个树对象
        - git read-tree：读取树对象
        > 使用上层指令时，对于没有变动的文件，git也会一并存储到树对象中，以保存目录结构的完整。其哈希值不变。
1. commit：将树对象和一些提交相关的元信息保存并串联起来
    - git commit-tree：输入指定的树对象、父commit、元信息（commit message等）创建一个新的commit对象
1. 标签：一个固定的哈希值和引用名
1. 分支：其本质就是记录在.git/refs中的若干个commit及其引用名
    - 当前仓库即HEAD引用：大部分情况下，HEAD存储的都是一个分支的名称（也就是引用名）。但少数情况下，当仓库成为分离（detached）状态时，HEAD文件是一个哈希值
1. 仓库同步的本质：将由commit节点组成的有向无环图进行同步
1. 其他：
    1. 存储：git最初向磁盘中存储对象是以松散的格式（文件都存全量）来存储，但会在特定时候将这些对象打包为一个packfile。打包过程将会把原有的全量文件之间进行比对，并以完整 + 差异的方式保存。
    1. 传输：git目前主流的方式有ssh、http(s)两种传输方式。无密码推拉方案有OAuth、SSh Key、GPG Key、token等等。
## 协作规范
1. 看所在公司的要求，当然，也看你自己的水平，但基本上：
    1. master/main禁止push，必须通过代码评审进行合并提交，来源是release、hotfix，并对每次提交打标签
    1. develop禁止push，必须通过代码评审进行合并提交，来源是各类fix、feature
    1. feature_xxx：用于开发
    1. release_xxx：从develop分支分离出来，用于对master上线前进行测试，建立后不应再接受push，测试成功后合并到master
    1. hotfix_xxx：热修复，从master来，回master去
## 常用命令及其应用场景
1. 基础操作：
    - git add
    - git rm
    - git stash
    - git commit
    - git checkout
1. 想同步：git pull
1. 想提交：git push
1. 想回到过去：git reset
1. 想回到另一个时间线的过去：git reflog
1. 想挑选修改：git cherry-pick
1. 想打标签：git tag
1. 想管理仓库：
    - 清理仓库：git gc
    - 查看大小：git count-objects
## 一些复杂的操作
1. 合并两个git仓库为1个，且保持双方提交
1. 彻底剔除从某次提交时加入的（大）文件
## 其他
- push 有的时候会失败，提示remote rejected之类的，commit some refs failed。并不确定是不是自己的问题。可以尝试git gc。如果还是不行，建议等一段时间，可能只是github的问题。
- git早期更像一个文件系统，即使是现在，很多底层命令依然保留。通常可以分为底层（plumbing）和上层（procelain）命令。
## 参考
- [Git Book中文版](https://git-scm.com/book/zh/v2)
    - [一定要看的Git Book原理部分](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)
- [Git 菜鸟教程](https://www.runoob.com/git/git-tutorial.html)