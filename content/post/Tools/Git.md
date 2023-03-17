---
title: "Git"
date: 2021-08-11T00:00:13+08:00
categories:
- 实用工具
tags:
- 实用工具
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/git.png
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
- .gitignore
- .gitsubmodule
## 核心原理
<img src="/images/tools/git-object-model.png"></img>
<img src="/images/tools/git-lifecycle.png"></img>
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
1. commit：将树对象（保存起来的blob）和一些提交相关的元信息保存并串联起来
    - git commit-tree：输入指定的树对象、父commit、元信息（commit message等）创建一个新的commit对象
1. 标签：一个固定的哈希值和引用名
1. 分支：其本质就是记录在.git/refs中的若干个commit及其引用名
    - 当前仓库即HEAD引用：大部分情况下，HEAD存储的都是一个分支的名称（也就是引用名）。但少数情况下，当仓库成为分离（detached）状态时，HEAD文件是一个哈希值
1. git status的本质：对比工作目录、暂存区（Index）、提交（HEAD）这三个顶级树对象之间的差异
1. 仓库同步的本质：将由commit节点组成的有向无环图进行同步
1. 其他：
    1. 存储：git最初向磁盘中存储对象是以松散的格式（文件都存全量）来存储，但会在特定时候将这些对象打包为一个packfile。打包过程将会把原有的全量文件之间进行比对，并以完整 + 差异的方式保存。
    1. 传输：git目前主流的方式有ssh、http(s)两种传输方式。无密码推拉方案有OAuth、SSh Key、GPG Key、token等等。
    1. .gitignore文件：使用标准的glob模式（shell中使用的简化正则表达式）进行递归匹配。建立gitignore文件嘴贱的办法是去github上下载所开发项目的。例如：
    ```shell
    # 忽略所有的.dll
    *.dll
    # 但是跟踪opengl32.dll
    !opengl32.dll
    # 以/开头则禁止递归，只忽略和.gitignore同级的build文件夹
    /build
    # 忽略递归下，任何名为bin的文件夹
    bin/
    # 忽略任何log/下的log文件，但一个*对于log/err/0.log无效
    log/*.log
    # 忽略任何log/目录及其子目录下的任何log
    log/**.log
    ```
## 协作规范
1. 看所在公司的要求，当然，也看你自己的水平，但基本上：
    1. master/main禁止push，必须通过代码评审进行合并提交，来源是release、hotfix，并对每次提交打标签
    1. develop禁止push，必须通过代码评审进行合并提交，来源是各类fix、feature
    1. feature_xxx：用于开发
    1. release_xxx：从develop分支分离出来，用于对master上线前进行测试，建立后不应再接受push，测试成功后合并到master
    1. hotfix_xxx：热修复，从master来，回master去
## 常用命令及其应用场景
1. 通配符说明：由于git有自己的正则表达式匹配方式，因此在git命令中使用通配符时，应当使用反斜杠（如\\*），否则将会以shell的方式进行解释，导致错误。
1. 引用：
    - branchName@{x}，可以引用对应分支的指定提交，其中xxx可以是
        - 数字：代表倒数第x次提交
        - 时间，指定时间前的提交
    - branchName^，可以引用对应分支的父提交
    - branch~x，x为数字，也是指倒数第x提交
    > windows下推荐加双引号，如"dev^"
1. 基础操作：
    - git clone URL localName：从远程克隆仓库拉取并存储到本地localName文件夹
    - git init：创建一个仓库（创建.git目录和必要的文件）
    - git add：提交未跟踪文件（untracked）、未暂存文件（modified unstaged）到暂存区（staged）
        - -i（交互式暂存）：决定每个文件的保存方式
        - --patch（交互式暂存）：允许部分暂存文件，用于希望将同一个文件的修改分多次提交的情况
    - git rm：删除一个自上次提交后未被修改文件，并且将该文件从跟踪文件清单中移除（等价于普通rm删除后再git add）
        - -f：删除一个自上次提交后被修改或已经暂存的文件，并从跟踪文件中移除。由于这类文件尚未进入commit树，不具有可恢复性，因此必须添加-f强制删除
        - --cached：从暂存区删除一个文件，并停止跟踪该文件，但该文件仍然保留在工作目录中
    - git mv：移动文件，也可以用于重命名
    - git stash：将当前所有未提交内容放入暂存栈
        - pop：将暂存栈栈顶弹出，同时将对应修改作用于工作目录（等于git stash apply + git stash drop）
        - list：查看当前暂存栈
    - git commit：-m提交，--amend合并到上一次提交
    - git checkout：切换分支、恢复文件到上次提交的状态
        - --patch：交互式部分恢复
        - -b branchName：创建分支并跳转过去
        - commitId fileName：相当于git reset --hard commitId fileName
    - git status：查看当前工作目录内的文件状态
    - git diff：查看**未暂存**的文件和上一次提交的区别
        - --staged：查看**已暂存**的文件和上一次提交的区别
        - --check：检查提交是否有空白字符
    - git log：查看每一次的提交日志、提交人等简要信息
        - -p：显示详细差异
        - --pretty=x：提供oneline、short、full、fuller等显示方式，还可以通过format自定义格式
        - a..b：显示在b中却不在a中的提交
        - a...b：显示不被a、b同时包含的提交
        - ^a b c：在b和c中却不在a中的提交，可以添加任意多的分支
    - git show tagname：查看某个tag对应的提交信息
    - git grep：能够从仓库的提交历史、工作目录、索引等各种对象中查找正则表达式
1. 想合并：git merge / rebase
    - 基本用法
        ```bash
        #--------- merge方式 ---------
        # 分支a：       <- 4
        #             /
        # 分支b：1 <- 2 <- 3
        git checkout b
        git merge a
        # 分支a：       <- 4 <-
        #             /        \
        # 分支b：1 <- 2 <- 3 <-- 5

        #--------- rebase方式 ---------
        # 分支a：       <- 4
        #             /
        # 分支b：1 <- 2 <- 3
        git checkout a
        git rebase b
        # 分支a：            <- 4
        #                  /
        # 分支b：1 <- 2 <- 3
        # 进一步合并
        git checkout b
        git merge a
        # 分支a：            <- 4
        #                  /
        # 分支b：1 <- 2 <- 3 <- 4'
        ```
    - 特别用法
        - git merge --abort：在合并冲突时使用，中断合并，恢复到合并前的状态
        - 合并冲突时，在终端环境下操作合并的办法
            ```bash
            # 查看合并问题
            git diff


            ```
    > 一般来说，本地操作下rebase是优于merge的。但一旦你要进行合并的提交已**被其他人使用**，那么使用merge。否则将会导致其他人陷入麻烦。
1. 想还原提交：git revert
    - git revert -m 1 HEAD：将HEAD还原到当前提交的第一父提交（-m 1），即合并时工作区所在的分支的提交
1. 想同步：git pull
1. 想提交：git push
1. 想回到过去：git reset（对工作区、暂存区、HEAD的不同处理）
    - --soft SHA-1 / HEAD~：通过哈希值，引用等方式，恢复HEAD到指定提交状态。以HEAD~为例，相当于取消刚刚的commit，但依然add了。
    - --mixed SHA-1 /HEAD~：默认情况，恢复HEAD、暂存区到指定状态。以HEAD~为例，相当于取消commit、取消add，但修改依然存在。
    - --hard：完全取消了从指定提交之后的所有提交（但是还是其实可以恢复的）
    - --mixed/--hard SHA-1 / HEAD~ fileName：恢复指定文件到指定提交（没有--soft）
    - --patch：交互式部分恢复
    > 注意在针对提交的移动时，reset会影响分支引用所指向的commit，它会导致当前分支的引用一起向前回溯。对比之下，checkout只会将HEAD移动。例如HEAD指向master，master指向某次提交，reset后，HEAD和master都移动了，checkout后，HEAD移动，master不动。
1. 想看操作记录：git reflog
    - 用于查看全部操作记录，可以记住对应操作前后的commitId，用于回到过去
1. 想挑选修改：git cherry-pick
    - git cherry-pick SHA-1：用于挑选其他分支的个别提交，并作用于有需要的分支
1. 想管理分支：git branch
    - -v：查看每个分支的最后一次提交哈希值和说明
    - -vv：比较分支和远程分支，给出更详细的说明
    - --merged：查看已经合并到当前分支的分支
    - --no-merged：查看未合并到当前分支的分支
    - -d branchName：删除指定分支
1. 想打标签：git tag
    - -a tagname -m "xxxx"：创建一个tag和对应的说明
    - -a tagname SHA-1：对某次提交创建tag
    - git push origin tagName：提交某次tag到git上（标签默认不同步）
    - git checkout branchName tagName：检出某个分支上的某个tag
1. 想管理仓库：
    - git remote：
        - -v：查看远程仓库的简写和对应的url
        - add name URL：添加简写、url
        - set-url name URL：修改
        - show name：查看某个简写对应的远程仓库的状态
        - remove name：删除到某个远程仓库的连接，本地相关的跟踪分支和配置也会一并删除
        - rename a b：将简写a修改为b
    - 清理仓库：git gc
    - 查看大小：git count-objects
1. 想配置git：
    ```bash
    # 帮助手册
    man git-config
    # 姓名、邮箱
    git config --global user.name "rex"
    git config --global user.email rex@hahaha.com
    # 默认文本编辑器
    git config --global core.editor emacs
    # 设置外部的合并和比较工具
    git config --global merge.tool code
    # git mergetool
    ```
## 一些复杂的操作
1. 合并两个git仓库为1个，且保持双方提交（即子树合并）
    - 复杂的方法一
    ```bash
    # 第一步
    # 添加一个远程仓库，引用名称为another_dir
    git remote add another_dir https://github.com/xxxx
    # 也可以是本地
    # git remote add another_dir file://D:/Projects/xxx
    # 获取内容(但不能直接作用于当前分支)
    git fetch another_dir

    # 第二步
    # 创建一个本地分支映射到远程分支，并切过去看看
    git checkout -b another_dir_branch another_dir/master

    # 第三步
    # 回到主分支，进行合并
    git checkout master
    # 将该分支写入到子目录another_sub_tree
    git read-tree --prefix=another_sub_tree -u another_dir_branch

    # 第四步（可选），即使进行了第五步，也还可以进行第四步
    # 根据需要，还可以切回对应分支，再获取些更新
    git checkout another_dir_branch
    git pull

    # 第五步
    # 正式的合并，压缩提交
    git checkout master
    git merge --squash -s recursive -Xsubtree=another_sub_tree another_dir_branch
    ```
    - 方法二：**简单粗暴**，先将远程仓库修改一下，再直接合并
    ```bash
    # another_dir中，将所有内容移动到子文件夹内
    mkdir another_sub_tree
    mv * ./another_sub_tree
    git add .
    git commit -m "move to sub dir"

    # 主项目中
    git remote add another <URL>
    git pull another
    # 允许合并无关历史
    git merge another/master --allow-unrelated-histories
    ```
1. 修改某次历史提交的提交信息
    ```bash
    # 使用git rebase
    # 例如修改倒数第3次提交
    git rebase -i HEAD~3
    # 编辑弹出的文本
    # ...

    # 保存
    git commit --amend
    git rebase --continue
    ```
1. 彻底剔除从某次提交时加入的（大）文件
    - filter-repo：基于python的一款工具，pip安装git-filter-repo。
    ```bash
    git filter-repo --invert-paths --path "/you/large/file"
    git gc --prune=now
    git push --force
    ```
1. 查找问题代码的引入提交
    - git提供二分查找的办法：git blame & git bisect
1. 子模块：git submodule
    - 父仓库会将子模块视为一个整体管理，而并不管理其内部的提交。即当子仓库有更新时，父仓库的后续提交，可以不更新所使用的子仓库的提交的引用
    - git submodule add URL：添加子模块仓库
    - git submodule init：子仓库默认并不下载，需要单独init
    - git submodule update：恢复子仓库到父仓库当前提交所使用的状态
    - git clone --recurse-submodules url：相当于git submodule init & update
1. 创建git离线更新包
    - git bundle create
## 其他
- push 有的时候会失败，提示remote rejected之类的，commit some refs failed。并不确定是不是自己的问题。可以尝试git gc。如果还是不行，建议等一段时间，可能只是github的问题。
- git早期更像一个文件系统，即使是现在，很多底层命令依然保留。通常可以分为底层（plumbing）和上层（procelain）命令。
- git add \*和git add .的区别：由于\*在shell中有特殊含义，因此前者实际由shell进行通配并传给git，而.在shell中没有特殊含义，即后者是真正的添加所有文件。
## 参考
- [Git Book中文版](https://git-scm.com/book/zh/v2)
    - [一定要看的Git Book原理部分](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)
- [Git 菜鸟教程](https://www.runoob.com/git/git-tutorial.html)