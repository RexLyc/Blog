---
title: "Lua：合集篇"
date: 2023-04-06T21:11:24+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Lua系列
- 合集篇
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/Lua-Logo.png
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
    -- [[
        跨行注释
    -- ]]

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

    -- 匿名定义函数
    testFunc = function(key,val) return key.."="..val end

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

    -- 面向对象
    Shape = {area = 0}
    -- 冒号定义和点号定义的最大区别就是不需要传递对象实例自身
    function Shape:new(objet,side)
        object = object or {}
        -- self相当于this指针，设置元表
        setmetatable(object, self)
        self.__index = self
        side = side or 0
        self.area = side * side
        return object
    end
    function Shape:print()
        print("面积为：",self.area)
    end
    -- 冒号调用
    myShape = Shape:new(nil,10)
    myShape:print()

    -- 继承的实现很简单，在new的内部调用父类的new
    Circle = Shape:new()
    function Circle:new(object,side)
        object = object or Shape:new(object,side)
        setmetatable(object,self)
        self.__index = self
        self.area = math.pi*side*side
        return object
    end
    ```
1. 元表
    1. 表table虽然能完成数据存储，但是不能直接提供表级别的操作。为此Lua引入了元表，使得表也能具备用户编写的特定行为能力。
    1. 基本语法
        ```lua
        -- 绑定元表
        mytable = setmetatable({},{})
        -- 获取元表
        print(getmetatable(mytable))

        mytable = setmetatable({},
            {
                -- 提供__index用于元表成员访问
                __index = { something = "something"
                    ,someFunc = function() print("haha") end
                },
                -- 提供__newindex用于元表添加成员
                __newindex = {},
                useless='233'
            })
        
        -- 理解__index的必要性
        print(mytable.useless) -- nil
        print(mytable.something)
        ```
    1. Lua对table的检索实际上有三步：
        1. 从原始表中查找该元素
        1. 表中没有，则查看是否有带__index定义的元表，没有返回nil
        1. 有元表，元表中的__index方法是一个表，则**重复上述步骤**。如果是一个函数，则调用函数并返回
        > 注1：如果是不存在的元素，但是元表提供了__newindex，则使用__newindex（存储到元表或调用函数），否则存储到表
        > 注2：对__index表中的元素进行写入，会在原始表中创建新的同名元素，并不会修改元表中__index表的元素值
    1. 元表的其他常见操作：
        1. rawset：在元表__newindex内支持将键值绑定到原始表
        1. rawget：不调用任何元表方法来获取键值（即获取原始表值）
        1. 元方法（自定义来扩展表的功能，相当于C++的运算符重载）
            | 方法 | 运算符或含义 |
            | --- | --- |
            | \_\_add | + |
            | \_\_sub | - |
            | \_\_mul | * |
            | \_\_div | / |
            | \_\_mod | % |
            | \_\_unm | -负号 |
            | \_\_concat | ..连接 |
            | \_\_eq | == |
            | \_\_lt | < |
            | \_\_le | <= |
            | \_\_call | 把该表作为"仿函数"调用时 |
            | \_\_tostring |  表的输出行为 |
1. 协程
    1. 协程的基本特点是，可以由用户程序控制放弃执行，所有协程共用线程。Lua中提供了对协程的支持，对于协程的介绍，可以查看参考中的相关介绍。
    1. 相关接口：coroutine.create、resume、status、wrap、running、yield
1. 错误处理
    1. assert函数：用来检验条件
    1. error函数：终止当前函数并带着错误信息返回
    1. pcall、xpcall：安全的调用函数，如果抛出异常，后者甚至可以查看堆栈
1. 其他补充
    1. Lua的面向对象实现，核心是理解元表metatable和元方法__index
        - 为什么要设置元表：Lua只是在用表和元表的关系在模拟类和实例之间的关系，是一种原型模式，设置元表，意味着元表就是原始表的类型
        - 为什么要设置__index：
            1. Lua只是绑定了原始表和元表，但是元表中的数据成员并未属于原始表，对于该数据成员的查找，需要按照查询规则进行
            1. 先查询原始表，发现没有这个成员变量，然后去查元表，元表必须定义了__index才能查询到
        - 如何实现private：通过local声明 + 闭包
    1. Lua的函数调用，并不会检查形参和实参是否一致，多余的参数会被舍弃，不足的参数会置nil作为局部变量（local）
    1. 类型：nil、boolean、number、string、function、userdata（C数据结构）、thread、table
        1. 特别的：
            - 未定义变量的类型都是nil，置nil等价于删除变量
            - table作为一种关联数组，下标**默认从1开始**，但是你也可以向0，甚至负数位置写入数据
            - table也支持关联容器式的访问、支持成员对象式访问、甚至也支持作为多维数组
            - table也可以添加函数
                ```lua
                module = {}
                function module.func()
                    print('hello')
                end
                ```
    1. 可变参数
        ```lua
        -- 使用{...}和#
        function avg(...)
            result = 0
            -- 将可变参数输入到table
            local arg = {...}
            for i,v in ipairs(arg) do
                result = result + v
            end
            -- 通过#操作符取数量
            return result / #arg
        end

        -- 使用select('#',...)，select(n,...)
        function nothing(...)
            -- 获取总数（不包括nil）
            for i = 1, select('#', ...) do
                -- 获取第i个，注意从1开始
                local arg = select(i, ...)
                print(arg)
            end
        end
        ```
    1. 运算符：
        1. 算数运算符：^乘幂、//整除
        1. 关系：~=不等于
        1. 逻辑：and、not、or
        1. 其他：..连接、#取长度
    1. 迭代器：
        ```lua
        -- 无状态迭代器：迭代器、迭代总数、迭代器起始，每次都从外部传递迭代器状态
        -- 例如ipairs的一种可能实现
        function iter(a, i)
            i = i + 1
            local v = a[i]
            if v then
                return i, v
        end

        function ipairs(a)
            return iter, a, 0
        end

        -- 有状态迭代器：迭代状态保存在内部，适合更复杂的迭代场景，一般使用闭包
        function myIter(collection)
            local index = 0
            local count = #collection
            return function()
                index = index + 1
                if index <= count then
                    return collection[index]
                end
            end
        end

        for item in myIter({'hello','world'}) do
            print(item)
        end
        ```
    1. 模块与包
        1. require函数
            ```lua
            -- 加载和使用
            require("module")
            module.someFunc()

            -- 加载并赋予别名
            local m = require("module")
            m.someFunc()
            ```
        1. 加载位置：require的加载机制靠环境变量LUA_PATH、LUA_CPATH，优先用前者
        1. loadlib函数
            ```lua
            -- 加载
            local f = assert(loadlib('/usr/local/lib/youlualib.so','something'))
            -- 初始化
            f()
            ```
    1. 垃圾回收
        1. Lua采用自动内存管理。实现了基于标记清楚的垃圾回收器。其不同版本实现的内容略有不同。比如5.1是三色标记法。
        1. 可以手动调用collectgarbage函数，控制垃圾回收器。
## 库推荐
### 内建库
1. debug
    - debug.debug、debug.traceback
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

## 参考
1. [Lua Tutorials](http://lua-users.org/wiki/TutorialDirectory)
1. [菜鸟教程Lua](https://www.runoob.com/lua/lua-basic-syntax.html)
1. [协程-廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1016959663602400/1017968846697824)
1. [Lua源码分析之垃圾回收GC](https://blog.csdn.net/fwb330198372/article/details/104263213)
1. [理解lua中的self](https://www.cnblogs.com/zhaoqingqing/p/3912782.html)
1. [Step By Step(Lua面向对象)](https://www.cnblogs.com/orangeform/archive/2012/07/06/2421656.html)