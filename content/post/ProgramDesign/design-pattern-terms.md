---
title: "设计模式-术语"
date: 2022-01-22T14:17:41+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 施工中
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/design-pattern.svg
---
软件工程中常有和设计模式相关的一些额外的术语，在这里进行一个整理。
<!--more-->
# 对象
1. O/RM（Object Relational Mapping，对象关系映射），将对象和关系型数据库进行绑定，以对象来表示关系数据。
    1. VO & PO，最基本的两种对象：
        1. VO（Value Object，值对象）：通常用于业务层之间（层内、各层之间）传递数据，可以和表对应，也可以不对应。由用户以new创建。
        1. PO（Persistent Object，持久对象）：向数据库添加或删除数据时产生的，仅存在于和数据库连接时，即仅存在于数据访问层。一定和表结构对应，需要实现序列化接口。
    1. DO（Domain Object，领域对象）：DDD（Domain-Driven Design，领域驱动设计）的术语，指抽象出来的业务实体。可以拥有含有业务逻辑的方法。
        - 和PO的主要区别：是否需要进行持久化。
    1. VO（View Object，显示层对象）：用于向前端输出的展示对象。
    1. TO（Transfer Object，传输对象）：用于在应用程序之间传输的对象。
    1. QO（Query Object）：查询对象，将所有支持用于查询的字段进行封装的对象。对应着SQL语句中的where子句。
    1. BO（Business Object，业务对象）：代表一个完整的业务流程，封装有业务逻辑的java对象，内部往往包含多个PO、VO（Value Object）进行业务操作。
        - 可以认为和DO等价。
    1. DAO（Data Access Object，数据访问对象）：夹在业务层和数据库资源中间，提供数据库的操作。例如Spring框架中的各种Mapper。
    1. DTO：（Data Transfer Object，数据传输对象）：用于前端和后端之间传输实际需要的内容的对象，一般只是一个表的部分字段。如果这个对象对应了界面的展示，那它就是VO（View Object）。
        - 但是：设计上并不推荐VO、DTO两者直接等同。因为数据应当可以和表现形式之间解耦（即相同的数据，也可以使用不同的方式进行展示）。
        - 和DO的区别：DO是完整的业务实体，但并不一定允许展示层查看所有字段、方法。
    1. POJO：符合Java Bean规范的简单对象。

# 函数式编程

# 