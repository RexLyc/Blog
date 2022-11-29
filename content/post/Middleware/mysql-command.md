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
## C - Create增
1. 用已有数据创建表
    ```sql
    -- MySQL写法
    CREATE TABLE temp_record SELECT * FROM record LIMIT 10;
    ```
## R - Retrieve查
1. 对列的值进行翻译
    ```sql
    -- 关键字 case when then else end
    SELECT
	( CASE WHEN 列名 = "条件1" THEN '翻译名1' ELSE '翻译名2' END ) AS 新列名
    FROM 你的表名;
    ```
## U - Update改
1. 用b表数据更新a表
```sql
-- MySQL
UPDATE a
INNER JOIN b
ON a.id = b.id
SET a.content = b.content; 
```

## D - Delete删


## 混合
1. 用查询结果插入
```sql
INSERT INTO my_table(column1) SELECT column1 from table2;
```
## 其他
- 命令行
    1. 登录指定的MySQL数据库
    ```bash
    mysql  -u用户名 -p密码 -hIP地址
    ```
    1. MySQL数据库导出（.sql文件）
    ```bash
    # 导出指定服务器，指定数据库，指定表的数据
    mysqldump -u用户名 -p密码 --skip-lock-tables \
        你的库名 你的表名 > ./dump.sql
    ```
    1. 数据导出为csv（需提前设置secure_file_priv）
    ```sql
    -- MySQL写法
    SELECT * FROM temp LIMIT 10 INTO OUTFILE '/tmp/test.csv'
        FIELDS TERMINATED BY ','
        OPTIONALLY ENCLOSED BY '"'
        LINES TERMINATED BY '\n';
    -- 如出现权限禁止，则需要设定secure_file_priv为保存路径
    -- 并重启MySQL
    ```
- 