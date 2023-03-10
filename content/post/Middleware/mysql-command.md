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
## DML
1. 更新：update
    - 用b表数据更新a表
        ```sql
        -- MySQL
        UPDATE a
        INNER JOIN b
        ON a.id = b.id
        SET a.content = b.content; 
        ```
1. 插入：insert
    - 用查询结果插入
        ```sql
        INSERT INTO my_table(column1) SELECT column1 from table2;
        ```
1. 删除：delete

## DQL
1. 基本流程顺序是：FROM、ON、JOIN、WHERE、GROUP BY、HAVING、ORDER BY、LIMIT、SELECT，流程中的每一步都会产生一个临时的虚拟表格，但是只有运算后满足条件的才会进入下一步产生的表格中[<sup>1</sup>](#参考)
1. 对列的值进行翻译
    ```sql
    -- 关键字 case when then else end
    SELECT
	( CASE WHEN 列名 = "条件1" THEN '翻译名1' ELSE '翻译名2' END ) AS 新列名
    FROM 你的表名;
    ```
1. having子句
    ```sql
    -- having子句的目标是提供对分组、聚合之后的结果的筛选能力，相对的where只能用在聚合和分组之前
    SELECT 列名1, SUM(列名2) AS 聚合函数结果, COUNT(1) FROM 表名 WHERE 列名3=XXX GROUP BY 列名4 HAVING 聚合函数结果 > 1
    ```
1. 各种连接
    ```sql
    -- 左、右外连接（左右外连接）
    SELECT * FROM A LEFT JOIN B ON A.KEYA=B.KEYB;
    SELECT * FROM A RIGHT JOIN B ON A.KEYA=B.KEYB;
    -- 左、右内连接通过判断非NULL实现
    SELECT * FROM A LEFT JOIN B ON A.KEYA=B.KEYB where B.KEYB IS NOT NULL
    SELECT * FROM A RIGHT JOIN B ON A.KEYA=B.KEYB where A.KEYA IS NOT NULL

    -- 内连接（MYSQL默认内连接）
    SELECT * FROM A INNER JOIN B ON A.KEYA=B.KEYB;
    SELECT * FROM A JOIN B ON A.KEYA=B.KEYB;

    -- 外连接用UNION（UNION默认去重）
    SELECT * FROM A LEFT JOIN B ON A.KEYA=B.KEYB
    UNION
    SELECT * FROM A RIGHT JOIN B ON A.KEYA=B.KEYB;
    ```
> 不同语言支持的SQL查询标准不同，当语法不兼容的时候，考虑用其他办法


## DDL
1. 修改表：alter
    - 删除某列
        ```sql
        -- 
        alter table xxx drop column xxx;
        ```
1. 创建表：create
    - 用已有数据创建
        ```sql
        -- MySQL写法
        CREATE TABLE temp_record SELECT * FROM record LIMIT 10;
        ```
1. 删除表：drop

## DCL
1. revoke
1. grant

## 坑
1. count：count(*)或count(1)都不会跳过null，但如果count(column)，即指定列名，则会跳过null

## 其他
- 表空间优化
    - optimize命令，优化表，有使用限制，且不同存储引擎的实现不同，如InnoDB将会重建表
    ```sql
    -- 优化表A（此时磁盘剩余控件必须大于A所需的表空间）
    optimize table A;
    ```
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
## 参考
1. [MySQL的执行顺序](https://www.jianshu.com/p/88c1b5e19cd8)