---
title: "Lua：合集篇"
date: 2023-04-06T21:11:24+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Lua系列
- 合集篇
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/Lua-Logo.jpg
---
据说Lua是一个非常好用的C/C++内嵌脚本语言，在系统编程和游戏领域都有一定的用武之地。本篇总结学习一下。
<!--more-->
## 基本
1. 看Lua安装后的example：
```lua
-- 函数也是一等公民
a=print
print(type(a))

-- 跨行字符串
a = [[
    hello
    world
]]

-- ''内可以包含""而不需要转义，反之亦然
a = ' \' " '
b = " \" ' "

-- 变量没有类型，变量的值是有类型的
a=123
b='c'
print(type(a))
print(type(b))

-- 不要使用以下划线_开头的变量名，避免和内置变量冲突，但单个的下划线_常用来作为哑变量
print(_VERSION)

-- 多赋值
a,b,c,d,e = 1,2,'three','four',5
-- 多赋值用于交换
a,b=b,a

-- 用..运算符完成字符串拼接，字符串接数值类型
a,b,c='hello','world',233
print(a..b..c)

-- io.write，不换行输出
io.write('hello ')
io.write('world')

-- 用{}创建table
a={}
b={1,2,"3"}
-- 创建后添加属性
a.name='haha'
-- 可以以成员或者map方式访问
print(a.name,a['name'])

-- if then else elseif end
if a==1 then
    print('1')
else
    print('not 1')
end

-- 条件赋值
b=(a==1) and "1" or "not 1"
b=(a~=1) and "not 1" or "1"
-- while循环、repeat循环
while a~=5 do
    a=a+1
    break
end

repeat
    a=a+1
until a==5

-- for循环：begin,end,step
for a=1,6,3 do
    print(a)
end
-- for循环：in 迭代器
for key,value in pairs({1,3,5,7}) do
    print(key,value)
end

-- 函数定义，支持多返回值
function myFunc()
    print('in myFunc')
    return 1,2,3
end
a,b,c=myFunc()

-- 变量具有默认全局作用域，除非使用local限定
b=1
function myFunc()
    local b="local var"
    a="?"
end
myFunc()
print(a,b)

-- 导入外部库
require('iuplua')
```
## 库推荐
### 内建库
1. math
    - math.abs、math.pi
1. string
    - string.find、string.match
1. table
    - table.insert
1. input/output
    - io.open、io.close、io.flush
    - file:close、file:seek
1. os
    - os.clock、os.execute
### 外部库
1. 日志
1. 网络
1. GUI
1. 性能

## 参考
1. [Lua Tutorials](http://lua-users.org/wiki/TutorialDirectory)
1. [菜鸟教程Lua](https://www.runoob.com/lua/lua-basic-syntax.html)