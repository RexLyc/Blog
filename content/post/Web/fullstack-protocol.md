---
title: "全栈学习：协议篇"
date: 2022-10-09T14:45:21+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 全栈
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/fullstack.jpg
---
本文整理在全栈开发过程中的一些经典协议和常见问题。
<!--more-->
## TCP/IP协议族
1. IP
1. UDP
1. TCP
    1. 异常情况处理
    1. 优化：
        1. 高并发
        1. 端口数量限制
1. 单播、组播、广播
## HTTP
## 编码
### 文字编码
1. 编码处理情况
    1. 代码
    1. 编译器
    1. 传输
    1. 展示
        1. 前端
        1. Qt
    1. 数据库
        1. mysql
    1. 操作系统
        1. Linux：$LANG
1. 常见编码：
    1. UTF
    1. GBK
### 多媒体编码
1. BASE64
### 条形码&二维码
1. 二维码
    1. 起源
    1. 基本原理
    1. 各版本
## RESTful
1. 概述：RESTFUL结构风格是互联网中HTTP接口经常采用的一种接口设计风格
1. 关键词：
    - 路径：又称为终点（Endpoint），restful架构中认为一个路径代表一种资源，因此路径中不应出现动词
    - http方法：代表对指定路径进行的操作
    - IETF：国际互联网工程任务组Internet Engineering Task Force，公开性质的民间国际团体
1. 常用方法
    1. GET：对应select
    1. PUT：对应update、insert
    1. POST：对应create
1. 其他方法
    1. HEAD
    1. DELETE
    1. OPTIONS
    1. CONNECT
    1. PATCH
    1. TRACE
1. 常见状态码
| 码值 | 英文名称 | 具体含义 |
| --- | --- | --- | --- |
| 200 | OK | 服务器成功返回请求数据 |
| 201 | Created | 常用于POST、PUT请求已成功，资源已保存 |
| 202 | Accepted | 服务器已经接收到请求，但不保证处理状态 |
| 204 | No Content | 服务器已经处理请求，但没有什么可以内容需要反馈，也不需要刷新页面 |
| 205 | Reset Content | 服务器已经处理请求，客户端需要刷新相关页面 |
| 206 | Partial Content | 当客户端标头中使用Range来请求范围数据时，使用此响应返回对应的范围数据 |
| 400 | Bad Request | 客户端请求有误，服务器不会处理 |
| 401 | Unauthorized | 实际是unauthenticated，意味着客户端未能认证自己的身份 |
| 403 | Forbidden | 客户认证了身份，但是不具备该请求所需的权限 |
| 404 | Not Found | 资源不存在，或者不便于向客户端公开 |
| 
> 注：HTTP响应码大致分为1xx（信息响应）、2xx（成功响应）、3xx（重定向消息）、4xx（客户端错误）、5xx（服务器错误）
1. HTTP扩展内容：
    - WebDAV：Web Distributed Authoring and Versioning，是一种允许客户端远程更新服务器内容的扩展，常用于实现增删改网页元数据（作者、创建时间）、复制移动网页、锁定文档禁止多人编辑等。
    - CalDAV：WebDAV的日历扩展
    - CardDAV：WebDAV的通讯录同步扩展
## 参考
- [tcp 为什么要三次握手，两次不行吗？为什么？ - 小林coding的回答 - 知乎](https://www.zhihu.com/question/429915921/answer/2682855827)
1. [RESTFUL Tutorial](https://restfulapi.net/)
1. [HTTP响应状态码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)
1. [RESTful API设计指南](https://www.ruanyifeng.com/blog/2014/05/restful_api.html)