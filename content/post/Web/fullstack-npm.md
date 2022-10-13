---
title: "全栈学习：NodeJS与NPM"
date: 2022-08-26T10:36:36+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 全栈
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/fullstack.jpg

---
Node.JS是基于Chrome V8开发的JS服务器端运行环境，NPM则是Node.JS的包管理器。
<!--more-->
## 概述
&emsp;&emsp;开发JS应用一般都会选择安装Node.JS进行开发。而安装过程中，也会一并安装NPM（Node Package Manager）。使用NPM能够方便的进行包的管理，项目开发、发布。

## NPM实用总结
1. 项目结构
    | 目录/文件 | 内容 |
    | --- | --- |
    | /build/ | 构建目标、构建脚本 |
    | /config/ | 一些配置脚本 |
    | /node_modules/ | 本地安装的包 |
    | /src/ | 源代码 |
    | /static/ | 静态资源文件 |
    | .babelrc | babel配置文件 |
    | index.html | 生成的首页入口 |
    | package.json | 项目配置 |
    | package-lock.json | npm5后添加，用于锁定版本号 |
    > 注：版本号约定X.Y.Z。X代表大变动，不向下兼容；Y代表新功能，但向下兼容；Z代表bug修补。注意package-lock将会完全锁定版本，不会自动更新。如果没有该文件，则会自动更新同一个X下的最新版本。
1. 重要配置
1. 实用命令
    | 命令 | 含义 |
    | --- | --- |
    | npm install xxx | 安装包xxx到当前项目 |
    | npm install xxx -g | 安装包xxx到全局环境 |
    | npm install npm -g | 升级npm |
    | npm list -g | 查看全局安装的包 |
    | npm uninstall xxx | 删除包 |
    | npm search xxx | 搜索包 |
1. 镜像
    - 使用cnpm
        ```bash
        # 指定从淘宝镜像下载、安装cnpm
        npm install -g cnpm --registry=https://registry.npm.taobao.org
        # 此后一直使用cnpm代替npm
        cnpm install xxx
        ```
    - 使用淘宝镜像
        ```bash
        # 单次使用
        npm install xxx --registry=https://registry.npm.taobao.org
        # 永久配置
        npm config set registry https://registry.npm.taobao.org
        # 还原
        npm config set registry https://registry.npmjs.org/
        ```
    - 注意：cnpm和npm的区别
        1. 本质上都是一种包管理器，而cnpm原生使用国内的镜像
        1. cnpm和npm无法完美混用，可以的情况下尽量使用npm，配置有效的镜像
## 一些库：
### babel
- 概述：目的是通过ES6到ES5的翻译，让ES6项目也能用于不支持ES6的浏览器
- 参考：[官方文档](https://babeljs.io/docs/en/)
### vue
- 概述：一个前端框架，支持双向绑定等
- 参考：
    - [vue config/index.js配置详解](https://blog.csdn.net/qq_31964019/article/details/106186776)

### electron
- 概述：一个用于生成跨平台程序的node.js库
- 实用工具：
    1. electron-forge：打包工具
    1. electron-builder：打成安装包的工具
- 参考：
    - [ElectronJS中获取GPU信息](https://www.imangodoc.com/199335.html)
    - [electron-builder打包优化](https://zhuanlan.zhihu.com/p/379467469)
## Node.JS实用总结
1. 版本变更
1. 

## 参考资料
- [菜鸟教程](https://www.runoob.com/nodejs/nodejs-tutorial.html)
- []()