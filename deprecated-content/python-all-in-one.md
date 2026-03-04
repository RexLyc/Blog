---
title: "Python：合集"
date: 2022-10-08T15:13:43+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Python系列
- 合集
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/python.jpg
---
Python作为一种脚本语言，在互联网和AI技术发展浪潮中都抓住了机会，目前应用非常广泛。本文整理了Python使用中的经典问题。
<!--more-->
## 历史
&emsp;&emsp;1989年冬季的圣诞节期间，荷兰人Guido van Rossum为了打发时间，决心编写一个更好用的脚本语言。最初的Python由C语言实现，可以调用C库。Python在开发过程中充分调动了社区的积极性，逐步创建了网站、基金会，获得了高速发展。
## 包和发行版
### 包管理器
    1. Pip：Python包的通用管理器，全称为**P**ip **I**nstall **P**ackages。只能用来管理Python包，其安装的包通常都是发布在Python Package Index（PyPI）上面。
    1. Conda：一个开源、跨平台的包和环境管理工具，可以用来管理任意语言的包，跟踪依赖，从这点来看，conda更像apt、yum。并且conda还支持对虚拟环境的管理，比如同时安装多个版本的python，而pip则需要借助virtualenv或venv等进行支持。
### 发行版
1. Anaconda：Anaconda公司开发的，包含一系列预先建立和配置好的包的发行版。
1. Miniconda：基本上是一个用来安装空的conda环境的发行版，仅包含conda和conda的依赖。
## PyPy
## 坑
1. pyinstaller部署
    - [pyinstaller打包含有socketio的flask项目的安装与使用](https://blog.csdn.net/qq_23518283/article/details/100514584)
## 参考
- [Anaconda与conda、pip与conda的区别](https://zhuanlan.zhihu.com/p/379321816)