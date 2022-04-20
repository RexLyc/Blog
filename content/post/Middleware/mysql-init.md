---
title: "MySQL：开坑篇"
date: 2022-01-24T19:22:25+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/mysql-logo.png
---
MySQL是当前一种主流的关系型数据库，本系列从原理和高性能方面入手进行学习
<!--more-->
## 历史
和想象中不同的是，MySQL的发展其实已经有很多年的历史。
- 1979年，程序员Monty Widenius为一个名为TcX的小公司打工时，用Basic设计了一个报表工具。之后又以C重写移植到UNIX平台。该工具是一个很底层且仅面向报表的存储引擎，名为Unireg。
- 1983年，Monty遇到了David Axmark，两人相见恨晚，开始合作运营TcX。公司在为一些超市做数据管理工具。此时他们开发了一个利用索引顺序存取数据的程序，也就是ISAM（Indexed Sequential Access Method）存储引擎核心算法的前身。
- 1990年，TcX公司的客户中有人要求提供SQL支持。Monty首先向将mSQL结合到自己的存储引擎中，但是测试后并不令人满意。MySQL就此诞生。
- 1995年，MySQL的第一个内部版本发行。1998年发行了第一个正式版本。但此时仍然非常弱小，不支持事务操作、子查询、外键、存储过程、视图等。
- 1999年，MySQL AB公司成立。合作开发了Berkeley DB引擎，支持事务处理。
- 2000年，改进ISAM，命名为MyISAM。
- 2001年，InnoDB成为MySQL的存储引擎备选方案。
- 2008年，MySQL AB被Sun收购。Sun则随后被Oracle收购。
- 2010年，MySQL 5.5发布，该版本非常经典，加强了企业级的各种特性。此版中InnoDB正式称为MySQL的默认存储引擎。
## 安装
- Linux/Ubuntu 18.04
```bash
sudo apt install mysql-server
mysql_secure_installation # 进行一些安全配置
mysql -u root -p # 登录root账户
# 输入密码
mysql>> # 开始数据库操作
        ...
```
- 可能出现的问题
    1. 安全配置之后无法登录，提示ERROR 1698
        - 目前发现，使用如下写法，是可以通过的
        ```bash
        # 即添加sudo
        sudo mysql_secure_installlation
        sudo mysql -u root -p
        ```
        - ***未知原因？***
    1. 无法远程访问，可参考[MySQL远程连接失败(错误码:2003)](https://blog.csdn.net/weixin_43025071/article/details/88603053)。
        - 检查端口、防火墙
        - 检查mysql库，user表，host & user字段：host应当是符号“%”才允许任意ip访问。
        - ***仍然失败？***
- 推荐一个命令行工具mycli

## 发展趋势
1. 存储计算分离
1. 超快扩缩容
1. 超大存储
1. 高可用

## 参考资料
1. [MySQL数据库的发展历史](https://www.cnblogs.com/joyfulcode/p/12683009.html)