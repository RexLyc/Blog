---
title: "边学边用linux-内存管理"
date: 2023-03-10T14:19:43+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
math: true
---
内存管理也是操作系统需要提供的核心功能之一，本文讲述Linux在这方面的相关内容。
<!--more-->

## 内存一致性模型

## 段页式虚拟内存

## Buddy
1. 目标：解决内存碎片中的外部碎片问题。在以页为单位的内存申请中，寻找最小的分配方式。
1. 基本原理：
    - 把所有的空闲页放到11个链表中，每个链表分别管理大小为1，2，4，8，16，32，64，128，256，512，1024个页的内存块。
    - 申请的时候，分配最小能满足大小的页，剩下的空余部分，分拆到其他链表中，如申请7个页，则从8中拿走一个，并将其多余的1个页放到第一个链表中
    - 页面释放的时候，如果相邻页面页是空闲的，则尝试组成更大的连续空闲页，返回到链表中

## Slab
1. 目标：解决内存碎片中的内部碎片问题，用于解决小对象频繁申请释放内存的效率问题
1. 历史：1994年在Sun系统中被提出(The Slab Allocator: An Object-Caching Kernel Memory Allocator, Jeff Bonwick (Sun Microsystems))，Slab是一种内存分配器，通过将内存划分不同大小的空间分配给对象使用来进行缓存管理，应用于内核对象的缓存。
1. 基本原理：
    - 基本流程
        - 首次申请小对象时，会构造一个kmem_cache内存池结构，并从buddy分配的页面，构造slab结构，将其加入该种类对象的内存池，创建并返回一个小对象
        - 此后申请小对象时则会遍历内存池，寻找合适大小的内存池，如果没有则继续申请内存，构造slab
        - 此后即使小对象释放内存，该页面也不一定会立刻返回给buddy
    - slabs链表：内存池内部会有slabs_full、slabs_partial、slabs_empty三种链表，分别保存全部占用、部分占用、未使用的slab，优先从部分占用中分配，未使用的链表在内存紧张时可能返回给系统
    - 着色：考虑到cache line的存在，对slabs链表中的对象分配，在跨slab时，一般会使用一定长度的对齐，避免对象的内存跨越了多个cache line，容易造成伪共享问题
1. 分配的对象大小：和buddy类似，也是11种大小
    - 从$2^3$到$2^11$个字节，此外还有96、192两个特殊的
    - 一个slab是内存池向系统申请内存的最小单位，根据所需要的大小，从一页到多页不等。
1. 其他：
    - 可从/proc/slabinfo查看slab对象分配情况

## 参考
1. [Linux内存管理之slab 1：slab原理（+buddy伙伴系统）](https://blog.csdn.net/lqy971966/article/details/112980005)
1. [linux内核slab机制分析](https://www.jianshu.com/p/95d68389fbd1)
1. [Linux内存管理(2) - buddy系统](https://blog.csdn.net/jasonchen_gbd/article/details/44023801)
1. [内存分配[四] - Linux中的Slab(1)](https://zhuanlan.zhihu.com/p/105582468)