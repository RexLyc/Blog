---
title: "Wheel：好用轮子合集篇"
date: 2023-02-19T22:44:36+08:00
categories:
- 计算机科学与技术
- 轮子
tags:
- 轮子
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/wheel.png
---
不要重复造轮子！本篇记录一些非常厉害好用的轮子，即一些类似于awesome-xxx的实用工具。
<!--more-->
## 跨语言
1. thrift：一个可以生成多语言的服务开发框架，可以很轻松的完成客户端-服务端开发。[Apache Thrift官方链接](https://thrift.apache.org/)

## 网站
1. dokuwiki：好用开源的简易wiki，这里直接记录一下部署步骤：
   1. 安装docker，推荐用官方脚本
   2. 部署dokuwiki-docker，推荐使用[bitnami打包的版本](https://hub.docker.com/r/bitnami/dokuwiki)，如果不满意可以在其基础上修改Dockerfile重新build
   3. 推荐一些插件
      1. bootstrap3 template
      2. add new page
      3. markdown
2. 指令
   ```bash
   # 下载compose yml
   curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/dokuwiki/docker-compose.yml > docker-compose.yml
   
   # 自定义修改yml
   # ...

   # 后台启动
   docker-compose up -d
   ```
3. 小坑：
   1. 需要对时区进行设置设置，分为修改docker-compose.yml和修改容器内php文件两步
      ```yml
      version: '2'
      services:
         dokuwiki:
            # 添加TZ=Asia/Shanghai
            # 这一步能修改容器内系统时间时区
            environment:
               - TZ=Asia/Shanghai
      ```
      ```php
      # 在/bitnami/dokuwiki/conf/local.protected.php内添加
      # 如果没有则创建
      <?php
      date_default_timezone_set("Asia/Shanghai");
      ```
   2. CSV插件支持无效，对于表格的支持要再想想办法
   3. Firefox使用时会报安全问题
   4. 用Semantic Plugin、Bootstrap 3 Template实现悬停预览内部链接页面时，如果PHP版本在8及以上，会报出一个渲染函数参数数量不匹配的问题（子类重写时少了一个参数）。需要手动修改该文件并进行替换
      ```php
      /** /opt/bitnami/dokuwiki/inc/parser/xhtmlsummary.php */
      /** 添加最后的 $returnonly = false */
      public function header($text, $level, $pos, $returnonly = false) {
         /** ... */
      }
      ```
      ```bash
      # docker容器内容替换
      docker cp ./xhtmlsummary.php xxxx:/opt/bitnami/dokuwiki/inc/parser/xhtmlsummary.php
      ```

## 图形编程
1. Manim：非常流行且强大的Python绘图工具。Youtube、bilibili上都有很多博主在使用。可以创建好看且流畅的动画。用来做教学，演示算法流程再好不过。[仓库链接](https://github.com/3b1b/manim)