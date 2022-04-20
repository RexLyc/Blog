---
title: "Mysql：SQL指令篇"
date: 2022-02-22T13:22:21+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/mysql-logo.png
---
本章以MySQL的SQL指令为主，也会结合其他的DBMS。
<!--more-->
## C

## R

## U
1. 用b表数据更新a表
```sql
-- MySQL
UPDATE a
INNER JOIN b
ON a.id = b.id
SET a.content = b.content; 
```

## D