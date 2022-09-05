---
title: "Vue3系列开坑篇"
date: 2021-07-22T17:06:17+08:00
categories:
- 计算机科学与技术
- Vue3
tags:
- 系列开坑
- Vue3
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/vue.png
---
&emsp;&emsp;前端是世界上最伟大的职业（迫真）。Vue3是国人（尤雨溪）主导的优秀前端框架，简单好用，童叟无欺。快进来看一看。
<!--more-->
&emsp;&emsp;Vue3的教程其实非常多，去bilibili上搜索就能搜索到非常多，因此本系列将会从实际案例出发，讲述一些Vue3的使用方式。

## 版本
&emsp;&emsp;由于vue2.x和vue3.x是隔离的两个版本。面向未来考虑，本系列将只针对vue3.x。截止本文发布时的版本为vue3.2.1。
## 前置知识
&emsp;&emsp;大概了解什么是HTTP（超文本传输协议，Web开发最重要的通信协议）、HTML（超文本标记语言，构成了页面的基本结构）、CSS（层叠样式表，美化界面）、Javascript（前端最常用的开发语言之一）、DOM（文档对象模型），掌握一门服务器端开发语言就可以了。另外如果会一点点网络通信的知识就更完美啦。
## 安装
1. 引入vue的方式有很多种。事实上大部分类库、框架都有这几种引入方式。
    1. 在页面上直接引入链接。
        ```html
        <script src="https://unpkg.com/vue@next"></script>
        ```
    2. 手动下载
    3. 使用npm（Nodejs包管理器）安装
    4. 使用vue-cli进行配置，直接获得一个完整的vue3前端工程。**[推荐]**
2. npm和vue-cli的安装。
    1. nodejs安装（自带npm）：去[Nodejs官网](https://nodejs.org/zh-cn/)下载安装包。
        - 安装中可选项主要有是否安装必要的扩展内容，用于支持如通过C++、Python编译部分内容的场景。我选择了安装。*不确定是否安装成功*
        - 相关扩展内容的手动安装步骤存在于[nodejs本地模块的python&vs构建工具](https://github.com/nodejs/node-gyp#on-windows)
    2. **[可选]**更换npm使用的源为淘宝镜像源。（也可以找其他镜像）
        ```bash
        npm config set registry http://registry.npm.taobao.org #全局切换
        npm get registry #查看当前源
        ```
    3. 安装脚手架vue-clie
        ```bash
        npm install -g @vue/cli
        vue upgrade --next
        ```
        - 注意区分@vue/cli、vue-cli，他们是脚手架的不同版本，相互之间隔离。
    4. 查看安装情况
        ```bash
        node --version
        npm --version
        vue -V
        ```
        - windows下可能需要修改cmd、powershell的部分权限。根据提示查询即可。
3. 使用脚手架创建项目
    ```bash
    cd /path/to/your/project
    vue create YourProjectName
    ```
    - 按照需要选择配置，不熟悉的话可以默认。
## 运行
&emsp;&emsp;在使用脚手架的前提下。运行一个vue3项目是很简单的。如对默认的空项目，只需到package.json同级目录下，执行
```bash
npm run serve #serve名称可修改，具体参照package.json中的配置
```
## 打包
&emsp;&emsp;根据package.json中关于build的配置，一般来说直接执行脚本即可完成用于部署的构建。如对默认空项目，在package.json同级下执行
```bash
npm run build
```
&emsp;&emsp;此时内容将会生成到dist/目录下。需要使用一个http服务器来进行对网页的服务。最简单的方法依然是使用npm，执行
```bash
npm run -g  serve #-g的意思是全局安装，而不是在当前项目中
serve -s dist #指定为dist目录做http服务器
```
- 额外注意
## 实用第三方库
1. 组件库：Element-Plus
2. 图形库：Pixi.js
2. 更多参考：[Github上的awesome系列中的前端内容](https://github.com/dypsilon/frontend-dev-bookmarks)
## 本系列暂定版本
1. Nodejs：14.17.4(LTS)
    - npm：6.14.14
2. @vue/cli： 4.5.13
## 参考资料
[Vue.js官方网站](https://v3.cn.vuejs.org)
