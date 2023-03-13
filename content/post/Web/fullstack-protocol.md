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
1. OSI各层都有自己逻辑上的业务，底层暂略
    - 应用层：为应用程序提供服务
    - 表示层：数据格式转换和加密
    - 会话层：建立、维持、管理会话
1. IP
1. UDP
1. TCP
    1. 特殊状态：
        - PSH：表示该数据不等数据包填充满，直接发送
        - URG：不走缓冲区，直接发送
        - RST：第三次握手失败，如果此时接收到数据，则会回复RST，强行断开连接
    1. 异常情况处理
    1. 优化：
        1. 高并发
        1. 端口数量限制
1. 单播、组播、广播
## HTTP
1. 使用最为广泛的应用层协议之一，建立在TCP上，以协议、地址、端口、路径表示一个HTTP接口
1. HTTP头
1. HTTP数据体
1. 坑：
    1. 二进制数据不能直接填充到文本类型的数据体中，由于编码问题，不能保证所有的二进制数据得到正确编码（可能被转为一个问号?），对于需要填入到json格式内的二进制数据，可以考虑转为BASE64编码
1. 大版本的主要变化
    - 1.0
    - 1.1
    - 2.0
    - 2.1
1. 流水线
    - 非流水线模式
    - 流水线模式
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
| 405 | Method Not Allowed | 该资源不支持该方法 |
| 408 | Request Timeout | 由服务器在空闲链接上发送，代表因客户端长期无请求，服务器将关闭连接 |
| 409 | Conflict | 代表此请求和服务器上的资源状态相冲突 |
| 413 | Payload Too Large | 请求实体大于服务器限制 |
| 414 | URI Too Long | URI长于服务器要求 |
| 415 | Unsupported Media Type | 不支持的媒体类型 |
| 429 | Too Many Requests | 用户在给定时间内发送了太多请求（限流） |
| 500 | Internal Server Error | 服务器内部错误 |
| 501 | Not Implemented | 不支持的请求方法，除了GET/HEAD是必须实现的，其他方法都可以返回501 |
| 502 | Bad Gateway | 网关/代理服务器从上游服务器得到了错误的响应 |
| 503 | Service Unavailable | 服务器未准备好 |
| 504 | Gateway Timeout | 网关超时 |
| 511 | Network Authentication Required | 由控制网络访问的拦截代理服务器生成，提示客户端需要验证 |
> 注1：HTTP响应码大致分为1xx（信息响应）、2xx（成功响应）、3xx（重定向消息）、4xx（客户端错误）、5xx（服务器错误）
> 注2：纠结是否仅使用标准HTTP响应码没有太大意义，以双方约定为主，实际上很多时候使用标准响应码反而会束缚住业务需求
> 注3：511常用于各类网络接入时，网络服务商提示客户端进行登录网络的身份验证
1. HTTP扩展内容：
    - WebDAV：Web Distributed Authoring and Versioning，是一种允许客户端远程更新服务器内容的扩展，常用于实现增删改网页元数据（作者、创建时间）、复制移动网页、锁定文档禁止多人编辑等。
    - CalDAV：WebDAV的日历扩展
    - CardDAV：WebDAV的通讯录同步扩展
## 参考
- [tcp 为什么要三次握手，两次不行吗？为什么？ - 小林coding的回答 - 知乎](https://www.zhihu.com/question/429915921/answer/2682855827)
- [RESTFUL Tutorial](https://restfulapi.net/)
- [HTTP响应状态码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)
- [RESTful API设计指南](https://www.ruanyifeng.com/blog/2014/05/restful_api.html)