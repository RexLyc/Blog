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
- 概述：一个用于生成跨平台程序的node.js库。分为主进程（main.js），渲染进程（各种html）
- 进程间通信：electron内最重要的事情之一，就是主进程到渲染进程之间的通信。这个通信一般使用ipcMain、ipcRenderer来完成，详见参考中的官网链接。需要注意的是只有在加载某个页面时指定preload文件，才能将通信的函数正常配置，区分node环境和浏览器环境，preload还是node环境，但是页面和页面引用到的js都是浏览器环境了。
- 实用工具：
    1. electron-forge：一个混合多种功能的cli，可以用于打包
    1. electron-package：打成安装包的工具，打包功能稍弱
    1. electron-builder：打成安装包的工具，打包功能更为强大
- 注意：
    1. npm设置代理或镜像的方式都不够稳定，怎么解决？
    1. electron-builder据说可以设置ELECTRON_MIRROR环境变量，未验证是否真的有效。否则builder也经常去拉github上的资源，容易失败。
        ```sh
        # windows下
        set ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/"
        ```
    1. files设置的打包时所需打包的文件，但是注意路径需要有一些变化。最好是通过from、to的方式指定清楚。
    1. ASAR文件存在一些限制，这其中最明显的是ASAR是只读的，因此不能将需要运行时修改的文件打包，此时应当使用"asarUnpack"选项，指定不想被打包的文件
    1. linux下图标最好使用icns文件指定
    1. linux下以deb包安装会出现权限问题，默认都是root文件，无法进行修改（比如动态调整配置文件）。个人认为最好的办法是将配置文件复制到用户目录下进行使用。
- 项目文件参考
```json
// package.json
```
- 参考：
    - [ElectronJS中获取GPU信息](https://www.imangodoc.com/199335.html)
    - [Electron使用electron-builder打包流程](https://segmentfault.com/a/1190000022763633)
    - [electron-builder打包优化](https://zhuanlan.zhihu.com/p/379467469)
    - [electron打包优化之路](https://segmentfault.com/a/1190000038574623)
    - [Electron 打包优化](https://www.jianshu.com/p/50043f485ec9/)
    - [使用electron-builder在windows上打包并自动更新](https://www.cxyzjd.com/article/weixin_34249678/89009487)
    - [官网：Electron 进程间通信](https://www.electronjs.org/zh/docs/latest/tutorial/ipc)
## Node.JS实用总结
1. 版本变更
1. 

## 参考资料
- [菜鸟教程](https://www.runoob.com/nodejs/nodejs-tutorial.html)
- 