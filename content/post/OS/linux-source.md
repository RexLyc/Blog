---
title: "边学边用linux-源代码篇"
date: 2022-12-10T12:54:37+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg

---
部分Linux源代码赏析，侧重在系统实现中用到的重要的数据结构和算法
<!--more-->
## 常见宏
- 内存相关
    ```c
    // 返回x所指向的内存区域的起始cacheline的边界地址
    #define L1_CACHE_ALIGN(x) __ALIGN_KERNEL(x, L1_CACHE_BYTES)
    ```
    ```c
    // 局部数据，将数据分配到程序.data段，起始位置对齐
    ____cacheline_aligned
    ```
    
    ```c
    // （下划线少了2个）全局数据，其他同上
    __cacheline_aligned
    ```
    ```c
    // 提供给list_head，计算所属结构体
    #define list_entry(ptr, type, member) \
	        container_of(ptr, type, member)

    // 从指定地址ptr，计算所属结构体的首地址
    #define container_of(ptr, type, member) ({                      \
            const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
            (type *)( (char *)__mptr - offsetof(type,member) );})

    // 通过0地址计算字段偏移量
    #define offsetof(TYPE, MEMBER)  ((size_t)&((TYPE *)0)->MEMBER)
    ```

## 数据结构
- 通用
    ```c
    // 通用双向链表的指针域
    struct list_head {
        struct list_head *next, *prev;
    };
    ```
    ```c
    // 哈希列表中的槽
    struct hlist_head {
        struct hlist_node *first;
    };

    // 哈希列表槽中的双向链表节点
    struct hlist_node {
        // pprev指向前驱的next指针
        struct hlist_node *next, **pprev;
    };
    ```

## 算法

## 参考
1. [Linux Kernel Map可视化](https://makelinux.github.io/kernel/map/)
2. [bootlin：一个整理linux、grub等多个项目全版本代码的工具网站](https://elixir.bootlin.com/linux/latest/source)
3. [Linux insides 中文翻译版（gitbook在线）](https://xinqiu.gitbooks.io/linux-insides-cn/content/)