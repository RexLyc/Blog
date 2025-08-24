---
title: "图解Linux内核 读书笔记"
date: 2025-01-01T16:28:33+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统
- 施工中
- 读书笔记
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/linux-kernel-pictures.jpg
draft: true
mermaid: true
math: true
---
本书提供了大量的插图，来学习Linux内核。
<!--more-->

> 推荐直接来这个网站看linux kernel：[bootlin](https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt)，这个页面是mm.txt的，不过版本较低（v5.0）。也有很多其他重量级开源项目。

> 本书有很多细节，汇编代码，因此也建议作为科普，工具书阅读。有需要的时候可以回来看看。

> 书中所使用的两个Linux版本，分别为3.10和6.2。如果某一个版本代码和书中对不上，就去看另一个版本吧。

## 概述和基础知识
1. 内核代码结构：
    - Documentation：文档
    - arch：和体系结构有关的，或者是其他模块中需要区分体系结构的内容
    - kernel：核心部分，包括进程调度、中断处理、时钟等，和体系结构相关的会放到/arch/xxx/kernel
    - drivers：驱动
    - mm：内存管理，同样也会有在/arch/xxx/xx下
    - fs：文件系统，一种文件系统拥有一种子目录
    - ipc：进程间通信
    - block：块设备管理
    - lib：内核空间下的通用函数库
    - init：内核初始化
    - firmware：由外部设备的芯片运行的固件程序
    - scripts：内核配置脚本
    - 其他（本书不涉及的）：net、crypto、certs、security、tools、virt（虚拟化）
1. 基础数据结构
    - Linux目前仍以C为主，所以其数据结构，以struct、和container_of等宏的方式形成，不像其他面向对象的语言提供的那种数据结构的形式
        ```c
        // 再复习一下 container of
        // typeof 是gnu c关键字
        #define container_of(ptr, type, member) ({   \
            const typeof(((type *)0)->member) * __mptr = (ptr); \
            (type *)((char *)__mptr - offsetof(type, member)); })

        #define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
        ```
    - 一对多的描述方式：将链表结构嵌入到有需要的数据结构中
        ```c
        // 方式一
        struct branch {
            struct list_head head;
            // other member
        };
        struct leaf {
            struct list_head node;
            // other member
        };

        // 用于串联的list_head
        struct list_head {
            struct list_head *next, *prev;
        };

        // 方式二，节省空间但有一些不便
        struct hlist_head {
            struct hlist_node *first;
        };
        struct hlist_node {
            // pprev是前一个hlist_node的next指针的地址
            // 即若prev => curr, curr->pprev == &prev->next
            struct hlist_node *next, **pprev;
            // other member
        };
        ```
    - 多对多的描述方式：将多对多联系，抽象为connection结构体，再嵌入到有需要的数据结构中
        
        示例，这只是一个选课场景（学生-老师）下的示例，表示c语言具备的多对多抽象能力。
        实际场景中，设备-设备处理程序，就是一个类似的多对多的关系。
        ```c
        // 书上的示例描述了一个老师-学生的多对多场景
        // 一个学生可以有多个老师，一个老师可以有多个学生

        // 具有同一个老师的某些学生
        struct s_connection {
            // 连接到下一个s_connection
            struct list_head node;
            // 指向一个学生
            struct student *student;
        };
        
        // 具有同一个学生的某些老师
        struct t_connection {
            // 连接到下一个t_connection
            struct list_head node;
            // 指向一个老师
            struct teacher *teacher;
        };

        // 因为师生关系一定是双向的，所以可以将两种联系合并, 代表一个学生-老师的联系
        struct connection {
            // 其他具有相同老师的connection
            struct list_head s_node;
            // 其他具有相同学生的connection
            struct list_head t_node;
            // 当前老师
            struct teacher *teacher;
            // 当前学生
            struct student *student;
        };

        // 最后，每个学生和老师的数据结构
        struct teacher {
            // 连接到connection中的s_node
            // 遍历该s_node的connection，可获得所有的student
            struct list_head head_of_student_list;
            // other member
        };
        struct student {
            // 连接到connecton中的t_node
            // 遍历该t_node的connection，可获得所有的teacher
            struct list_head head_of_teacher_list;
            // other member
        }
        ```
2. 设计模式：注意内核的设计方式，是面向对象的
    - 模板方法模式：Template Method。即开发者实现固定的接口，系统会根据流程进行调用。
    - 观察者模式：内核中，xxx_listener、xxx_notify
3. 中断
    
    广义的中断可进一步细分为中断（interrupt）和异常（exception）。更进一步的，中断分为可屏蔽和不可屏蔽，都是来自I/O设备的。异常则是程序主动进行的，包括陷阱、故障和终止。不论是哪种，CPU只会在一个指令执行完成后再检查，**不会在执行中检查**。
    
    ![中断处理流程](/images/book/linux-pic/idt.png)

    中断处理需要软硬件分工合作。中断控制器和CPU相连，单CPU架构和SMP架构中分别是PIC（可编程中断控制器），IOAPIC（高级可编程中断控制器）。CPU提供了处理的指令、以及相应的寄存器位来存储。

    区分两个概念：**中断处理程序**是指整个处理过程，从保护现场、处理、恢复现场。**中断服务例程**是其中的一部分，是专门处理产生中断的设备的相关逻辑的。中断服务例程涉及到两个关键的结构体：`irq_desc`、`irqaction`，是一对多，因为一个中断是可以被共享的。一个irq号对应一个`irq_desc`，会有通用的handler，而一个`irqaction`则代表一种设备更具体的处理，会有自己的handler供`irq_desc`中的handler调用。

    Top Half和Bottom Half，一些函数可看到th、bh的后缀，代表前半段、后半段。中断处理应当快速，头半段不能做复杂的处理。复杂逻辑应当使用工作队列、软中断，或启动单独的线程工作。

    注册`irq_desc`和`irqaction`的方式。
    ```c
    int request_threaded_irq(unsigned int irq, irq_handler_t handler,
        irq_handler_t thread_fn, unsigned long flags, const char *name, void *dev);
    
    int request_irq(unsigned int irq, irq_handler_t handler,
        unsigned long flags, const char *name, void *dev);
    ```

    这里有一个具体的例子，键盘和鼠标等外设，可以是共享相同irq（例如200）的设备，但是二者并不会直接拉起一次中断处理程序，会是由GPIO再发起一个irq（例如50），GPIO的设备将会负责相应的`irq_desc`的处理。键盘和鼠标只需要完成自己的对应200的，`irq_desc`，`irq_action`。共享中断需要该设备的驱动能够区分出来，是否是自己设备发出的，如果不能，那么不能进行共享。

    中断处理还有很多细节：比如中断处理时，又有新中断发生（一般来说是会继续处理最新的中断，如果有多个新中断，会丢失中间的）；是否还有软中断需要处理；中断处理结束后，需要返回内核态还是用户态等

    **软中断**：对于timer、tasklet等，内核支持一些软中断来完成这些事情。主要包括定时器，小任务，网络读写，块读写等。注意这里说的都是内核空间的事情，是内核可以使用的能力。

    从处理流程上来看，系统调用其实也是中断（异常）的一种。因此减少系统调用对优化是有一定作用的。

4. Linux的时间
    
    内核的时间功能分为两种：一个是作为时钟源，提供时间戳信息。另一个是提供时钟中断，以供一次性或周期性事件的触发。

    内核的时间单位：jiffy，滴答。以及```ktime_t```。

    核心的数据结构，```timekeeper```、```clocksource```、```clock_event_device```时钟事件。
    
    时钟源是有等级的，内核会选择一个作为“看门狗”，其他时钟源可以受此监督，如果某个其他时钟源误差过大，将会置为不稳定。时钟芯片有很多种，RTC、PIT、TSC、HPET、APIC Timer。

    内核维护的事件有多种，常见的有：REALTIME（系统时间，也是WALL TIME墙上时间）、MONOTONIC（非休眠时间）、BOOTTIME（启动时间，包括休眠）

    时钟中断是触发进程调度的最常见的情景之一。在中断章节中也可以，这种情况下，中断程序负责标记进程需要调度，并在中断返回时，内核态下检查标记在内核态进一步完成调度。

## 内存管理篇

### 内存寻址
   
广义的内存管理，也就是CPU所说的内存管理，其实是包括所有有效的连接在总线上的存储。换言之CPU访问的物理地址并不一定真的在RAM里。

既然内存空间包括多种存储设备，Linux系统将会把所有的这些映射到内存空间中，即MMIO（memory mapped io）。可以通过```/proc/iomem```。设备寄存器、显存等都可以是MMIO的一部分。不过这一点需要CPU架构的支持。

而且内存空间并不是连续的，会有一些用不到的空洞Hole。

内存管理，实际上需要维护内存介质（RAM + MMIO）、内存空间、虚拟内存**三者之间**的关系。其中前两者之间的映射，由BIOS完成。
![mmio](/images/book/linux-pic/mmio.png)

Linux中共有三种地址：虚拟地址（Virtual Address）、线性地址（Linear Address）、物理地址（Physical Address）。应用程序使用的是虚拟地址，虚拟地址通过分段机制（用户代码段、用户数据段、内核代码段、内核数据段）后就变为线性地址。内存管理单元MMU将会用分页机制，把线性地址转换为物理地址。Linux上虚拟地址和线性地址其实几乎相同（段描述符基准地址为0）。

MMU寻址部分就是多级页表的机制。32位和64位有一些区别。这里强调几点：寻址由MMU硬件完成，各级页表项所包含的地址都是物理地址。页框是指划分好的一块连续的物理内存，而页/页面是指对应页框大小虚拟内存。P.S.:可以再去[复习一下](https://blog.csdn.net/weixin_49342084/article/details/142773491)CR3寄存器（PDBR）、PTBR。页表的加载和寻址是从CR3赋值开始的，这一数据存储在每个进程的进程控制块task_struct中。

即使有了MMU，操作系统仍然需要完成虚拟地址到物理地址的映射的建立。就是说页表需要操作系统来设置，内核提供了大量的函数和宏来做这些事情。如果有需要，这里可以结合一些博客来学习，强烈推荐如[Linux Kernel直接映射区的构建](https://zhuanlan.zhihu.com/p/692536727)、[Linux Kernel内存管理之分页](https://zhuanlan.zhihu.com/p/661911303)。另外也可以考虑参考[Intel x86-64开发人员手册](https://www.intel.cn/content/www/cn/zh/content-details/858440/intel-64-and-ia-32-architectures-software-developer-s-manual-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4.html)，该手册内有很多图表值得一看。不看这些博客的话，简单看一下下面也可以。

目前分页最多的时候有5级页表，页全局目录PGD、页四级目录P4D、页上级目录PUD、页中级目录PMD、页表PT（第一级）。而且一般如果只有2级，则P4D、PUD、PMD的项数均为0，即10、0、0、0、10，最后页表内有12位物理地址偏移，总共32位。常规4K页面的分页下的一个内核直接映射内存区的页表建立方法如下：

> 内存直接映射区：**内核空间**中一个大的，连续的虚拟内存空间，他映射了部分或所有物理内存。

```c
/*
* 页表导航说明：
*
* pgd, p4d, pud, pmd, pte 均为指向各级页表项的虚拟地址指针。
* pfn（页框号）为物理页帧编号，此处为 0x12，对应物理地址 0x12000（即 0x12 << PAGE_SHIFT）。
*
* 本代码目标：为指定的物理页帧 pfn 建立对应的页表映射（线性地址空间中）。
*/

// 计算给定物理页帧在线性地址空间中的 PGD（Page Global Directory）索引
// 注意：(pfn << PAGE_SHIFT) + PAGE_OFFSET 将物理页帧转换为对应的线性地址（内核线性地址直接映射区）
// PAGE_OFFSET 是内核线性映射的起始虚拟地址偏移（如 0xFFFF888000000000 在 x86_64）
pgd_idx = pgd_index((pfn << PAGE_SHIFT) + PAGE_OFFSET);
pgd = pgd_base + pgd_idx;  // 获取该线性地址对应的 PGD 项指针

/*
* 在当前配置（通常为 4-level 分页但启用兼容模式或线性映射平坦）下，
* p4d、pud、pmd 层级可能被折叠或直接透传，因此偏移量为 0。
* 使用 p4d_offset/pud_offset/pmd_offset 获取下一级页表指针。
*/
p4d = p4d_offset(pgd, 0);
pud = pud_offset(p4d, 0);
pmd = pmd_offset(pud, 0);

/*
* 检查 PMD 项是否已存在且有效（即指向一个页表页）。
* 如果对应页表页未分配（_PAGE_PRESENT 位未设置），则需分配一个新页表。
*/
pte_ofs = pte_index((pfn << PAGE_SHIFT) + PAGE_OFFSET);  // 计算 PTE 索引（页内偏移）
if (! (pmd_val(*pmd) & _PAGE_PRESENT)) {
    // 分配一个位于低地址区域的物理页作为页表页（页表本身存储空间）
    pte_t *page_table = (pte_t*) alloc_low_page();

    /*
    * 构造 PMD 项值：
    *   - __pa(page_table): 获取 page_table 的物理地址
    *   - _PAGE_TABLE: 标志位，表示该 PMD 指向一个页表（而非大页）
    *   - __pmd(): 将整型值封装为 PMD 类型
    *   - set_pmd(): 安全地更新 PMD 项
    */
    set_pmd(pmd, __pmd(__pa(page_table) | _PAGE_TABLE));
}

/*
* 获取最终的 PTE（Page Table Entry）指针。
* pte_offset_kernel() 根据 PMD 和线性地址中的页内偏移计算出 PTE 位置。
*/
pte = pte_offset_kernel(pmd, pte_ofs);

/*
* 设置 PTE 项，建立最终的物理页映射：
*   - pfn_pte(pfn, prot): 将页帧号 pfn 与访问权限 prot 组合成一个 PTE 值
*   - set_pte(): 将生成的 PTE 值写入页表项
*/
set_pte(pte, pfn_pte(pfn, prot));

/*
* 至此，物理页帧 pfn 已成功映射到线性地址空间中对应的位置。
* 后续可通过 (pfn << PAGE_SHIFT) + PAGE_OFFSET 访问该物理页。
*/
```

不过注意虽然现代计算机已经开始64位了，但其实并不允许使用全部的64位寻址，而通常只使用48位（而且用户空间为高16位为0，内核空间高16位为1）。中间空洞的地址是非法的，因此实际上一共只能使用256T内存。

> 其他扩展阅读：[linux kernel pwn之ret2dir攻击学习](https://www.anquanke.com/post/id/185408)

### 物理内存的管理
    
> 联动一下博客中的：[边学边用linux-内存管理]({{<relref "/content/post/OS/linux-memory.md#Buddy">}})

概念：节点（node）、区域（zone）、非统一内存访问（NUMA，和传统SMP架构相对，以socket为区分，将CPU和内存分组为不同的node，一组CPU访问自己组内的内存更快）。可以在```lscpu```中看到cpu的分组信息。

BIOS提供了SRAT（System Resource Affinity Table）、SLIT（System Locality Information Table）两个表，用来确定系统资源亲和性和延迟的信息。系统会进一步用来控制CPU上进程的对应的物理内存申请。

而zone则是对node内的资源再进行划分。zonelist中存储的就是对node中的内存的划分。划分至少是出于兼容性的考虑，比如有些设备只能访问指定的部分，因此需要将这部分内存保留出来。

![node-zone](/images/book/linux-pic/node-zone.png)

内核分配内存时，每一个NUMA节点就会从节点保存的zonelist上寻找。如果有多个node且允许尝试其他node的内存，则需要维护一个更复杂的zonelist（维护所有node的所有zone）。注意不同的NUMA节点，其zonelist会略有差别。总的来说会按照优先本地，优先高位地址的顺序排列。

一页物理内存对应一个Linux中的```page```对象。在这个思路指导下，Linux管理物理内存实际上有三种模式：FLATMEM、SPARSEMEM、SPARSEMEM_VMEMMAP。区别在于对物理内存的认定，以及对page对象的管理方式不同，page对象和pfn（页框号）的转换方式不同。

内存配置情况，可以通过```/sys/firmware/memmap```查看，这里会列出每一段bios提供的物理内存段。但是注意其中并不是所有的部分都可以用作内存分配，有一些内存会预留给其他模块使用。这些不能用物理内存也称为hole。

- FLATMEM：把内存看作连续的，即使中间有上面说到的hole，这些hole也是有page对象对应的。显然会造成一些page对象的浪费。
- SPARSEMEM：将内存做切分，有效的部分分配若干连续的section，section内是若干page，无效的hole部分不再分配section&page。
- SPARSEMEM_VMEMMAP模式【理解存疑】：依然会为有效的部分分配若干的section，但是要求分配出来的page对象的地址位于虚拟地址连续的区间上。也就是说page对应的虚拟内存地址从一开始就是确定了的。不过只有活跃的部分才会得到真正的物理内存。这种模式下，对于某个物理页而言，其pfn对应的page对象的虚拟地址是```vmemmap + pfn```。

> 区分对内存连续性的要求，虚拟地址连续性是比较好满足的，但仍然有一些场景，比如使用DMA时，可能需要物理地址也连续。

![SPARSEMEM_VMEMMAP](/images/book/linux-pic/sparsemem_vmemmap.png)

内存申请管理一般有三个阶段：启动程序、memblock、buddy。启动阶段即grub程序，grub程序可以通过```mem```参数来限制内核可管理的内存上限。memblock也可以通过将内存块加入```reserve```数据组扣留一部分，最后才是buddy系统管理。对于操作系统而言，memblock是内存管理的第一个阶段，buddy系统会接替他的工作。

> 内存管理还有更多方案：比如huge tlb，但本书并未讨论。

buddy系统的名字恰如其实。buddy将内存分为不同大小的块，1页，2页，4页...1024页（对应4K、8K、16K...4M）共11个级别（order阶）。如果块的伙伴也是空闲的（实际上已分配出去的块，不再属于伙伴系统），就可以合并为一个更大的块。确定伙伴的规则包括：
1. 两个块相邻，且位于同一个zone
2. 每个块大小都是2的整数次幂。合并后也要是，所以两个快的阶要相同
3. 两个块的地址必须是$2^n$对齐的，合并之后第一个块的地址则需要是$2^(n+1)$对齐的

zone和page是上下层级的关系。完整的层级是section（内存初始化和热插拔单位）→zone（分配管理单元）→page（页）。zone内按照阶，存储了所有阶```free_area```，其中每个还分为可迁移和不可迁移等类型。具体如下图。

![zone_page](/images/book/linux-pic/zone_page.png)

页的申请和释放函数，是上面曾见到过的：```alloc_page/pages```,```free_page/pages```等。```alloc```函数在使用时有很多参数，包含优先选择的zone和其他影响内存分配的行为，比如分配的优先级（是否需要保持zone内的分配水位，更高优先级可以使用一些预留的内存）。

可以看出，buddy系统所能提供的物理内存，要么可以物理地址连续但不能超过4M，要么可以超过4M但物理地址不能保证连续了。


### 虚拟内存的管理
   
每一个进程的线性地址空间（虚拟地址）划分分为内核态和用户态。内核态起始位置就是之前见到过的宏，```PAGE_OFFSET```。而且内核空间实际上是进程间直接、或者间接共享的。可以理解为用户态空间互相独立，内核态空间共享。正因如此，用户空间的页表需要进程自行维护，是用户页表。而内核页表很多情况下是相同的，属于公共的部分。

在x86时期，空间有限，内核虽然一般有1G的线性空间，但是并不能直接映射1G的物理内存。一般只能直接映射一部分（896M），剩下的部分保留满足其他需秋，直接映射的部分叫```Low Memory```,剩下的部分是```High Memory```。注意x86时期，物理内存是可以超过4G的，但是线性空间只有4G。

所谓直接映射，就是映射后的虚拟地址和物理地址有直接关系，在前面的代码中也能看到：$va = vp + PAGEOFFSET$。此时映射状态（左侧物理内存，右侧线性空间）

![x86-va-pa](/images/book/linux-pic/x86-va-pa.png)

而等到了x86-64时代，线性空间足够大了，不再区分```Low/High Memory```。

具体来看，内核线性空间（从高地址到低地址）内部还分为若干区域：
- 32位：固定映射区、永久映射区、CPU Entry区、动态映射区、直接映射区
- 64位：-

![x86-va-space](/images/book/linux-pic/x86-va-space.png)
上图为32位

![x64-va-space](/images/book/linux-pic/x64-va-space.png)
上图为64位
> 64位可能有4、5级页表等不同情况，这里是5级的布局。

接下来介绍一下内核线性空间中的各个区：
1. 直接映射区：大小理论上是MAXMEM，不考虑```High Memory```的情况下，会一直映射到没有物理内存为止。映射完成后在运行期内不变，因此需要稳定存在的数据结构需要用直接映射区。
2. 动态映射区：其他区域多少都有限制，但动态映射区能满足各类需求。常见的```ioremap```都在此区域实现。由```get_vm_area```函数族来分配此区域空间。用红黑树管理。
3. 永久映射区：x86-64上已经不再有这一区域。内核使用```kmap```函数将一页物理内存映射到该区。```kmap```的参数就是page结构体，如果page对应的物理页在```High Memory```会占用永久映射区，如果不再，则返回直接映射下的虚拟地址。该区域也只能以页的单位来进行分配。实际上和“永久”并没有关系。可能会用于和一些设备通信的内存区域。
4. 固定映射区：内部分为若干小区间，每个区间有特定用途。值得提到的是其中有一个临时映射区，为每一个CPU准备了一些页，通过```kmap_tomic```等函数操作物理内存映射到该区域，申请释放都很快，适合临时使用。


#### 详解用户空间内存映射mmap
mmap其实并不陌生，用于将文件/设备映射进内存，后续可以项访问内存一样访问。但要注意mmap使用的一定是用户线性空间。函数原型如下
```c
void* mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```

其中```flags```有一些讲究了，比如MAP_PRIVATE、MAP_SHARED。前者采用COW策略（Copy On Write），对映射区的更新将会对其他映射了同一区域的进程不可见，也不会写回文件。后者则是共享所有更新，并且会写文件。这里提前说一下，共享是针对物理内存的，后面会详细展开。大的费雷上，mmap分为匿名映射（不由文件映射而来）和非匿名映射（有具体映射的文件/设备）

当然，由于mmap也可以映射设备，因此并不是所有对文件的操作都可以用，具体支持情况依赖于设备驱动。

用户线性空间是以```vm_area_struct```来描述一个一个的用户线性空间中的区域的。进一步整合到进程结构体中的`mm_struct`。
```c
/*
 * This struct defines a memory VMM memory area. There is one of these
 * per VM-area/task.  A VM area is any part of the process virtual memory
 * space that has a special rule for the page-fault handlers (ie a shared
 * library, the executable area etc).
 */
struct vm_area_struct {
	/* The first cache line has the info for VMA tree walking. */

	unsigned long vm_start;		/* Our start address within vm_mm. */
	unsigned long vm_end;		/* The first byte after our end address
					   within vm_mm. */

	/* linked list of VM areas per task, sorted by address */
	struct vm_area_struct *vm_next, *vm_prev;

	struct rb_node vm_rb;

	/*
	 * Largest free memory gap in bytes to the left of this VMA.
	 * Either between this VMA and vma->vm_prev, or between one of the
	 * VMAs below us in the VMA rbtree and its ->vm_prev. This helps
	 * get_unmapped_area find a free area of the right size.
	 */
	unsigned long rb_subtree_gap;

	/* Second cache line starts here. */

	struct mm_struct *vm_mm;	/* The address space we belong to. */
	pgprot_t vm_page_prot;		/* Access permissions of this VMA. */
	unsigned long vm_flags;		/* Flags, see mm.h. */

	/*
	 * For areas with an address space and backing store,
	 * linkage into the address_space->i_mmap interval tree.
	 */
	struct {
		struct rb_node rb;
		unsigned long rb_subtree_last;
	} shared;

	/*
	 * A file's MAP_PRIVATE vma can be in both i_mmap tree and anon_vma
	 * list, after a COW of one of the file pages.	A MAP_SHARED vma
	 * can only be in the i_mmap tree.  An anonymous MAP_PRIVATE, stack
	 * or brk vma (with NULL file) can only be in an anon_vma list.
	 */
	struct list_head anon_vma_chain; /* Serialized by mmap_sem &
					  * page_table_lock */
	struct anon_vma *anon_vma;	/* Serialized by page_table_lock */

	/* Function pointers to deal with this struct. */
	const struct vm_operations_struct *vm_ops;

	/* Information about our backing store: */
	unsigned long vm_pgoff;		/* Offset (within vm_file) in PAGE_SIZE
					   units */
	struct file * vm_file;		/* File we map to (can be NULL). */
	void * vm_private_data;		/* was vm_pte (shared mem) */

	atomic_long_t swap_readahead_info;
#ifndef CONFIG_MMU
	struct vm_region *vm_region;	/* NOMMU mapping region */
#endif
#ifdef CONFIG_NUMA
	struct mempolicy *vm_policy;	/* NUMA policy for the VMA */
#endif
	struct vm_userfaultfd_ctx vm_userfaultfd_ctx;
} __randomize_layout;

// 这段代码也要在旧一点的版本中才能找到，
struct mm_struct {
	struct {
        // 映射区域链表头
		struct vm_area_struct *mmap;
        // 映射区域红黑树
		struct rb_root mm_rb;
    }
    // ... 其他成员暂时忽略
}
```

mmap的实现细节，都在```do_mmap```函数中。

![do_mmap](/images/book/linux-pic/do_mmap.png)

调用流程中，`get_unmmapped_area`用来获取可用的线性空间（也就是书中所说的“坑”）。物理内存在书中则用“萝卜”指代。如果用户不指定`addr`的话，这一步会根据当前进程`mm_struct`对象中`mm_mt`字段来查找合适区域。如果指定了，并检查了该地址开始的线性空间长度足够则使用，否则忽略`addr`重新分配。

而`mmap_region`则进行具体映射。初始化映射对应的`vm_area_struct`，完成映射，将对象插入红黑树。

物理内存的使用情况，也就是mmap最终的效果主要是由对应的文件/设备提供的驱动决定的。这有几种分类
1. 驱动有自己的物理内存（比如MMIO下的显存），驱动可以使用ioremap将其映射到内核线性空间的动态映射区。
2. 驱动需要申请内存然后在做映射。这里根据需要还会分为是否要申请连续物理内存。
3. 驱动中的mmap不提供映射，由后续的内存访问异常，触发内核调用驱动的fault操作，申请物理内存page赋值给vm_fault字段。

在了解了以上这些之后，就能明白共享内存其实是共享物理内存。不同进程之间需要将同一段物理内存，映射到自己的线性空间内。

书中提到`/dev/mem`设备，这是一个影射了物理内存的设备，我们可以用mmap将此设备进行映射，并直接操作物理内存。当然这一操作由于危险性极高，很多情况下已经被禁止直接使用（对应的区域禁止映射）。如果要用的话，可能需要编写驱动，以MMIO的方式进行使用。

#### 内存管理进阶

本节讨论一些更复杂的问题。

1. **内存申请：**
   - 申请连续物理内存。在buddy系统之下，内核还维护了slab系统，现在有多个版本，slab、slob、slub。slab内部使用`kmalloc/kfree`。起到内存池的作用。受buddy的限制，最大4MB的连续物理内存。
   - 申请连续虚拟内存。使用`vmalloc`，申请一段一段的物理内存，让后映射到连续的线性空间段上。`vmalloc`的使用场景是为内核态准备内存，因此更新的是内核页表，而非进程的页表。其虚拟地址在内核的动态映射区。

   无论slab还是vmalloc，返回的都是虚拟内存。相比之下，grub、memblock、buddy申请和管理的则是物理内存。

2. **缓存**
   
   内存中的数据可以分为两种：页表数据和实际数据。
   
   - TLB缓存用于加速对页表的访问。TLB是比较特殊的，内核写页表不会通过TLB，而是直接写内存。所以所有页表项更新的情况下，TLB都需要刷新。
   - cache缓存用于加速对普通数据的访问。实际上MMIO也会被cache进行缓存。cache有很多的缓存策略，在x86架构下：
        - Strong Uncacheable：UC，读写都不经缓存
        - Uncacheable：也是UC，但可以用MTRR（Memory Type Range Register）将其变为WC
        - Write Combining：WC，允许CPU缓冲多个写操作，并在合适的时候一次性写回内存
        - Write Back：WB，读写都经过缓存
        - Write Through：WT，和WB类似，但是写操作也同时写内存
        - Write Protected：WP，和WB类似，但是每次写都会导致缓存失效
        
        需要区分明确是因为，并不是所有的场景都可以使用WB这种效率最高的情况，如果写内存有副作用（比如MMIO的内存，可能是设备的控制位），那就不能用缓存了。

        MTRR机制可以用来设置一段物理内存的缓存方式。BIOS一般已经配置了，可在`/proc/mtrr`文件查看。修改该文件可以更改缓存方式。MTRR有一定限制（硬件相关），所以又有了PAT（Page Attribute Table），粒度是页，可以按照页来精准控制内存的缓存属性。可以在`/sys/kernel/debug/x86/pat_memtype_list`文件中看到配置。

        内存和缓存的不一致问题，不仅在于CPU的读写，在DMA设备访问内存的情况下，也会出现不一致的情况。这种时候也需要刷新缓存。

3. **缺页异常**
   
   缺页异常实际上会有不同的种类，CPU提供两项信息：错误码和异常地址。其中错误码存储在栈中，引发缺页异常的虚拟地址存储在CR2寄存器中。

   整体上看错误有三种场景：
   - 程序逻辑错误：空指针、访问越界、违反权限（写了只读内存）
   - 访问地址未映射物理内存
   - TLB过时
   - COW（Copy On Write）等场景，内存没有写权限。
  
    缺页异常程序是`asm_exc_page_fault`，汇编程序【似乎在内核代码中不太好找到实际代码段】。负责保存现场，并收集error_code。最终错误码和异常地址会传递到`handle_page_fault`。

    ![handle page fault](/images/book/linux-pic/handle_page_fault.png)

    根据地址位于内核还是用户空间，分别调用不同的处理程序。
    
    地址位于内核空间的情况下：如果进程是用户态，那么只有`vmalloc`和`spurious`两种情况可以处理。因为vmalloc的内存分配情况存储在内核页表，使用了vmalloc申请的内存会缺页异常，需要将页表拷贝给进程页表。spurious则是指的TLB刷新不及时（内存已经变为可读写，但是TLB中仍只读）的情况，产生的虚假错误。各种`bad_area`函数用来处理其它的情况，如果发生缺页异常时，进程处于内核态，会尽量尝试修复错误，否则直接发送SIGSEGV给用户态进程。

    地址位于用户空间的情况下：核心目标就是为地址找到对应的vma（就是我们前面提到过的vm_area_struct）并映射内存。在确认vma和当前的操作权限匹配后，开始真正的缺页处理。这里还要分为三种情况
    1. 没有完整的物理内存映射，需要申请内存并映射。
    2. 映射存在，但是物理页被交换了，需要将其读到内存。
    3. 映射完整，内存可写，但是页表中权限是只读，写内存异常。这对应的也是前面的COW等场景。

    这里需要格外强调：用户空间虚拟内存访问权限分成两个部分。内存映射的权限（在`vma->vm_flags`中），以及页表的权限。前者是全集，在其之外的是错误，没有讨论余地。后者则是表示实际的访问权限，就是说页表中的权限可能发生变化，来适应对应的需求。另外这里所说的两个权限，都是再`vma`的结构体中（flag和prot）。而DMA访问内存实际上是用pte访问的，因此内存实际上一共有三个权限在共同工作。

    `handle_pte_fault`比较重要，是站在pte的角度，处理以上的这些问题。按照书上的内容，补充了一些注释。

    ```c
    static vm_fault_t handle_pte_fault(struct vm_fault *vmf)
    {
        pte_t entry;

        // 这一段if + else，是在处理第一种问题。等待后续分配并映射。
        if (unlikely(pmd_none(*vmf->pmd))) {
            /*
            * Leave __pte_alloc() until later: because vm_ops->fault may
            * want to allocate huge page, and if we expose page table
            * for an instant, it will be difficult to retract from
            * concurrent faults and from rmap lookups.
            */
            vmf->pte = NULL;
            vmf->flags &= ~FAULT_FLAG_ORIG_PTE_VALID;
        } else {
            /*
            * If a huge pmd materialized under us just retry later.  Use
            * pmd_trans_unstable() via pmd_devmap_trans_unstable() instead
            * of pmd_trans_huge() to ensure the pmd didn't become
            * pmd_trans_huge under us and then back to pmd_none, as a
            * result of MADV_DONTNEED running immediately after a huge pmd
            * fault in a different thread of this mm, in turn leading to a
            * misleading pmd_trans_huge() retval. All we have to ensure is
            * that it is a regular pmd that we can walk with
            * pte_offset_map() and we can do that through an atomic read
            * in C, which is what pmd_trans_unstable() provides.
            */
            if (pmd_devmap_trans_unstable(vmf->pmd))
                return 0;
            /*
            * A regular pmd is established and it can't morph into a huge
            * pmd from under us anymore at this point because we hold the
            * mmap_lock read mode and khugepaged takes it in write mode.
            * So now it's safe to run pte_offset_map().
            */
            vmf->pte = pte_offset_map(vmf->pmd, vmf->address);
            vmf->orig_pte = *vmf->pte;
            vmf->flags |= FAULT_FLAG_ORIG_PTE_VALID;

            /*
            * some architectures can have larger ptes than wordsize,
            * e.g.ppc44x-defconfig has CONFIG_PTE_64BIT=y and
            * CONFIG_32BIT=y, so READ_ONCE cannot guarantee atomic
            * accesses.  The code below just needs a consistent view
            * for the ifs and we later double check anyway with the
            * ptl lock held. So here a barrier will do.
            */
            barrier();
            if (pte_none(vmf->orig_pte)) {
                pte_unmap(vmf->pte);
                vmf->pte = NULL;
            }
        }

        // 继续处理第一种问题（此时pte为NULL）
        if (!vmf->pte) {
            if (vma_is_anonymous(vmf->vma))
                return do_anonymous_page(vmf);
            else
                /*
                非匿名映射，内部根据不同情况进行处理
                1. 读操作异常：do_read_fault
                2. 写MAP_PRIVATE映射的内存：do_cow_fault
                3. 写MAP_SHARED映射的内存：do_shared_fault

                总之最终都会回调vma->vm_ops->fault得到一页内存，再用finish_fault更新页表

                这里do_cow_fault会申请到一页新的物理内存（vmf->cow_page），初始内容是从之前的页vmf->page拷贝过来的
                */
                return do_fault(vmf);
        }

        // 处理第二种情况，加载交换出去的内存
        if (!pte_present(vmf->orig_pte))
            return do_swap_page(vmf);
        
        if (pte_protnone(vmf->orig_pte) && vma_is_accessible(vmf->vma))
            return do_numa_page(vmf);

        vmf->ptl = pte_lockptr(vmf->vma->vm_mm, vmf->pmd);
        spin_lock(vmf->ptl);
        entry = vmf->orig_pte;
        if (unlikely(!pte_same(*vmf->pte, entry))) {
            update_mmu_tlb(vmf->vma, vmf->address, vmf->pte);
            goto unlock;
        }

        // 处理第三种情况，写操作异常，没有写权限
        // 注意区分，上面是pte为NULL的流程，而这里pte是存在的。但是pte中缺少写权限，而禁止了这次访问。
        // PROT_WRITE且MAP_SHARED，调用相关函数修改权限为可写
        // PROT_WRITE且MAP_PRIVATE，是COW，申请新的物理内存，复制内容，更新页表
        if (vmf->flags & (FAULT_FLAG_WRITE|FAULT_FLAG_UNSHARE)) {
            if (!pte_write(entry))
                return do_wp_page(vmf);
            else if (likely(vmf->flags & FAULT_FLAG_WRITE))
                entry = pte_mkdirty(entry);
        }
        entry = pte_mkyoung(entry);
        if (ptep_set_access_flags(vmf->vma, vmf->address, vmf->pte, entry,
                    vmf->flags & FAULT_FLAG_WRITE)) {
            update_mmu_cache(vmf->vma, vmf->address, vmf->pte);
        } else {
            /* Skip spurious TLB flush for retried page fault */
            if (vmf->flags & FAULT_FLAG_TRIED)
                goto unlock;
            /*
            * This is needed only for protection faults but the arch code
            * is not yet telling us if this is a protection fault or not.
            * This still avoids useless tlb flushes for .text page faults
            * with threads.
            */
            if (vmf->flags & FAULT_FLAG_WRITE)
                flush_tlb_fix_spurious_fault(vmf->vma, vmf->address);
        }
    unlock:
        pte_unmap_unlock(vmf->pte, vmf->ptl);
        return 0;
    }
    ```

    COW有一个非常经典的应用场景：fork子进程。因为子进程需要继承父进程的很多信息，这部分信息复制实际上由`dup_mmap`完成。在复制的过程中，主要就是在做COW。

    ![fork-cow](/images/book/linux-pic/fork-cow.png)

    为了保证COW的效果，实际上父子进程的pte项中的权限都会降级。仔细想想这里其实会有一个问题。就是如果父进程此时想写这里的内存，那么COW的优化意义实际上就失效了，父进程必须复制这段内存（因为父进程要修改了），即使后面并子进程不需要写，这其实可能出现浪费。所以子进程先执行COW更合理。而且如果子进程执行新的程序，那么很多内存都不需要复制，出于这个考虑，内核有一个变量来控制子进程是否可以抢占父进程。


### 内存回收

前文在伙伴系统的讲解中，忽略了内存回收这个大问题。这个问题主要由`_alloc_pages_slowpath`完成。

进行回收时可能有几种情况：
1. 空闲内存足够，但是碎片过多，没有连续内存。这是需要移动并合并一些内存碎片，进行规整（`compact`）。
2. 空闲内存不足，需要释放一些已经被占用的内存,就是回收（`reclaim`）。典型的例子是mmap的内存，如果释放的话，就写回`swap`（匿名映射），或者写回文件（非匿名映射）

**扫描**是回收的第一步。扫描过程由`scan_control`结构体控制。其内容如下
```c
// code from kernel 6.2
struct scan_control {
	/* How many pages shrink_list() should reclaim */
	unsigned long nr_to_reclaim;

	/*
	 * Nodemask of nodes allowed by the caller. If NULL, all nodes
	 * are scanned.
	 */
	nodemask_t	*nodemask;

	/*
	 * The memory cgroup that hit its limit and as a result is the
	 * primary target of this reclaim invocation.
	 */
	struct mem_cgroup *target_mem_cgroup;

	/*
	 * Scan pressure balancing between anon and file LRUs
	 */
	unsigned long	anon_cost;
	unsigned long	file_cost;

	/* Can active folios be deactivated as part of reclaim? */
#define DEACTIVATE_ANON 1
#define DEACTIVATE_FILE 2
	unsigned int may_deactivate:2;
	unsigned int force_deactivate:1;
	unsigned int skipped_deactivate:1;

	/* Writepage batching in laptop mode; RECLAIM_WRITE */
	unsigned int may_writepage:1;

	/* Can mapped folios be reclaimed? */
	unsigned int may_unmap:1;

	/* Can folios be swapped as part of reclaim? */
	unsigned int may_swap:1;

	/* Proactive reclaim invoked by userspace through memory.reclaim */
	unsigned int proactive:1;

	/*
	 * Cgroup memory below memory.low is protected as long as we
	 * don't threaten to OOM. If any cgroup is reclaimed at
	 * reduced force or passed over entirely due to its memory.low
	 * setting (memcg_low_skipped), and nothing is reclaimed as a
	 * result, then go back for one more cycle that reclaims the protected
	 * memory (memcg_low_reclaim) to avert OOM.
	 */
	unsigned int memcg_low_reclaim:1;
	unsigned int memcg_low_skipped:1;

	unsigned int hibernation_mode:1;

	/* One of the zones is ready for compaction */
	unsigned int compaction_ready:1;

	/* There is easily reclaimable cold cache in the current node */
	unsigned int cache_trim_mode:1;

	/* The file folios on the current node are dangerously low */
	unsigned int file_is_tiny:1;

	/* Always discard instead of demoting to lower tier memory */
	unsigned int no_demotion:1;

#ifdef CONFIG_LRU_GEN
	/* help kswapd make better choices among multiple memcgs */
	unsigned int memcgs_need_aging:1;
	unsigned long last_reclaimed;
#endif

	/* Allocation order */
	s8 order;

	/* Scan (total_size >> priority) pages at once */
    // 补充：就是说priority是右移参数，越小的话，扫描的页数越多
	s8 priority;

	/* The highest zone to isolate folios for reclaim from */
	s8 reclaim_idx;

	/* This context's GFP mask */
	gfp_t gfp_mask;

	/* Incremented by the number of inactive pages that were scanned */
	unsigned long nr_scanned;

	/* Number of pages freed so far during a call to shrink_zones() */
	unsigned long nr_reclaimed;

	struct {
		unsigned int dirty;
		unsigned int unqueued_dirty;
		unsigned int congested;
		unsigned int writeback;
		unsigned int immediate;
		unsigned int file_taken;
		unsigned int taken;
	} nr;

	/* for recording the reclaimed slab by now */
	struct reclaim_state reclaim_state;
};
```

从结构体中可以看到。包含了需要回收的页数，已扫描的页数、已回收的页数等等信息。具体的回收函数`shrink_zones`是在一个循环中进行的，在某次调用后，可能出现的情况有：
1. 当已回收的页数大于需要回收的页数，函数成功退出
2. 已回收的页数不够，增加下次扫描的页数
3. 如果扫描页数最大化还是找不到足够的内存。可尝试不跳过active的页（默认跳过）再试一下。还不行就只能返回失败了

而这个回收函数，老内核调用`shrink_zones`，内部再调用`shrink_node`。高版本直接调用后者，也就是直接以node为单位了。可扫描的页都存储在一个LRU list上。不同版本中所在的位置页不同，老版本在`zone`中（`zone.lruvec`），新版本在`pglist_data.__lruvec`

`lruvec`实际上是一个链表数组，里面每一个元素都是一个链表，链表元素也就是一系列被扫描的页。链表的类型有多种：匿名页链表、文件页链表。并且还分为活跃、非活跃（最近一段时间是否被访问），每个链表内部还是按照LRU处理的。这些`lru`上的页都是内核申请的页。应用、驱动都不感知这些页的信息，只是能够使用而已，因此回收过程，实际上我们只要保证下一次再访问时内容正确，将他们暂时从物理内存移除是完全ok的。

不过当然，不是所有内存都可以放到lru链表中。比如在驱动中使用`alloc_pages`申请内存，驱动在使用完成后将他们释放掉（`free_pages`）。虽然这些内存还是buddy系统分配的，但是这些内存的生命周期由模块本身负责，内核并不能直接回收这部分。但是内核也为模块留了一个口子，就是`shrinker`，如果模块实现了回收方法，并注册到内核，在回收时，也会尝试由模块释放一些内存。

`shrink_lruvec`的逻辑比较清晰：
1. 计算各类LRU需要扫描的页数
2. 循环调用`shrink_list`，每次尝试一种类型的LRU链表

```c
// code from kernel 6.2
static void shrink_lruvec(struct lruvec *lruvec, struct scan_control *sc)
{
	unsigned long nr[NR_LRU_LISTS];
	unsigned long targets[NR_LRU_LISTS];
	unsigned long nr_to_scan;
	enum lru_list lru;
	unsigned long nr_reclaimed = 0;
	unsigned long nr_to_reclaim = sc->nr_to_reclaim;
	bool proportional_reclaim;
	struct blk_plug plug;

	if (lru_gen_enabled()) {
		lru_gen_shrink_lruvec(lruvec, sc);
		return;
	}

    // 计算各类LRU扫描数量
	get_scan_count(lruvec, sc, nr);

	/* Record the original scan target for proportional adjustments later */
	memcpy(targets, nr, sizeof(nr));

	/*
	 * Global reclaiming within direct reclaim at DEF_PRIORITY is a normal
	 * event that can occur when there is little memory pressure e.g.
	 * multiple streaming readers/writers. Hence, we do not abort scanning
	 * when the requested number of pages are reclaimed when scanning at
	 * DEF_PRIORITY on the assumption that the fact we are direct
	 * reclaiming implies that kswapd is not keeping up and it is best to
	 * do a batch of work at once. For memcg reclaim one check is made to
	 * abort proportional reclaim if either the file or anon lru has already
	 * dropped to zero at the first pass.
	 */
	proportional_reclaim = (!cgroup_reclaim(sc) && !current_is_kswapd() &&
				sc->priority == DEF_PRIORITY);

	blk_start_plug(&plug);
	while (nr[LRU_INACTIVE_ANON] || nr[LRU_ACTIVE_FILE] ||
					nr[LRU_INACTIVE_FILE]) {
		unsigned long nr_anon, nr_file, percentage;
		unsigned long nr_scanned;

        // 遍历每一种lru
		for_each_evictable_lru(lru) {
			if (nr[lru]) {
				nr_to_scan = min(nr[lru], SWAP_CLUSTER_MAX);
				nr[lru] -= nr_to_scan;

                // shink_list内有对inactive、active的分别回收
				nr_reclaimed += shrink_list(lru, nr_to_scan,
							    lruvec, sc);
			}
		}

		cond_resched();

		if (nr_reclaimed < nr_to_reclaim || proportional_reclaim)
			continue;

		/*
		 * For kswapd and memcg, reclaim at least the number of pages
		 * requested. Ensure that the anon and file LRUs are scanned
		 * proportionally what was requested by get_scan_count(). We
		 * stop reclaiming one LRU and reduce the amount scanning
		 * proportional to the original scan target.
		 */
		nr_file = nr[LRU_INACTIVE_FILE] + nr[LRU_ACTIVE_FILE];
		nr_anon = nr[LRU_INACTIVE_ANON] + nr[LRU_ACTIVE_ANON];

		/*
		 * It's just vindictive to attack the larger once the smaller
		 * has gone to zero.  And given the way we stop scanning the
		 * smaller below, this makes sure that we only make one nudge
		 * towards proportionality once we've got nr_to_reclaim.
		 */
		if (!nr_file || !nr_anon)
			break;

		if (nr_file > nr_anon) {
			unsigned long scan_target = targets[LRU_INACTIVE_ANON] +
						targets[LRU_ACTIVE_ANON] + 1;
			lru = LRU_BASE;
			percentage = nr_anon * 100 / scan_target;
		} else {
			unsigned long scan_target = targets[LRU_INACTIVE_FILE] +
						targets[LRU_ACTIVE_FILE] + 1;
			lru = LRU_FILE;
			percentage = nr_file * 100 / scan_target;
		}

		/* Stop scanning the smaller of the LRU */
		nr[lru] = 0;
		nr[lru + LRU_ACTIVE] = 0;

		/*
		 * Recalculate the other LRU scan count based on its original
		 * scan target and the percentage scanning already complete
		 */
		lru = (lru == LRU_FILE) ? LRU_BASE : LRU_FILE;
		nr_scanned = targets[lru] - nr[lru];
		nr[lru] = targets[lru] * (100 - percentage) / 100;
		nr[lru] -= min(nr[lru], nr_scanned);

		lru += LRU_ACTIVE;
		nr_scanned = targets[lru] - nr[lru];
		nr[lru] = targets[lru] * (100 - percentage) / 100;
		nr[lru] -= min(nr[lru], nr_scanned);
	}
	blk_finish_plug(&plug);
	sc->nr_reclaimed += nr_reclaimed;

	/*
	 * Even if we did not try to evict anon pages at all, we want to
	 * rebalance the anon lru active/inactive ratio.
	 */
	if (can_age_anon_pages(lruvec_pgdat(lruvec), sc) &&
	    inactive_is_low(lruvec, LRU_INACTIVE_ANON))
		shrink_active_list(SWAP_CLUSTER_MAX, lruvec,
				   sc, LRU_ACTIVE_ANON);
}
```

LRU链表的一些添加和移除的细节情况。
1. 访问页时，会导致active和inactive链表之箭的移动。为了优化，这个移动操作被批量化了，保存在LRU cache中。在扫描inactive list之前，`lru_add_drain`必须将这部分排空，加入到对应的LRU链表中，避免漏扫。
2. 页隔离，`isolate_lru_folios/isolate_lru_pages`用来将待扫描的页从所在的`LRU list`中删除，并加入到`folio_list`链表，这样接下来就不回被重复扫描了。当然也就能避免重复回收。

> **性能思考**：为什么隔离采用了删除的方式，而不是添加标记之类的方式。其实可以认为，删除的方式，锁的粒度很小，而且失败了的线程可以直接跳过，去隔离其他的页。另外，将待扫描的页统一到folio_list中，后续批量处理速度更快。

folio（英文原意，对开本）看起来是一个突然出现的概念。但其实一定程度上就是复合页（compound pages），比如某些情况下一个folio中包括的页数是2的整数次幂。注意folio这个概念是在逐渐取代page。但目前内核中仍然会同时存在：page、Compound page（复合页）、folio。folio和page大部分字段都是一致的。所以在高版本中，会用`isolate_lru_folios`，其实是在向folios做迁移。

隔离到了足够多的页之后，就可以开始回收了。`shrink_folio_list`在500行左右（有不少的注释）,这里不再贴代码，可以直接[点击链接](https://elixir.bootlin.com/linux/v6.2/source/mm/vmscan.c#L1651)。这里直接总结一下书中整理的10个步骤，也就是在循环中执行：
1. 从folio_list（隔离出来的页的列表）中取下一个folio
2. 如果folio正在写回，等待其写完，并重新插入folio_list尾部，下次继续循环处理
3. 检查folio的活跃程度，即上次扫描到现在，**访问了物理页的映射的数量**（也就是访问了的pte的数量，而不是访问的数量）。（使用这个数据是因为MMU硬件上是在PTE中提供一个访问标记，而不是访问次数）


## 课后问题
1. 进程控制块中包含了进程页表的基址，那进程控制块本身所在的虚拟内存，由谁的页表管理，以及其内存基址如何存储？如何避免套娃问题？
   
   涉及到内核启动、分页机制、进程管理的启动

2. 物理内存是否可以热插拔，热插拔是否会引起直接映射区的重新映射？将被拔出的内存中的数据如何保存？
3. malloc、free的底层原理，他们是如何操作brk这个系统调用的
   【TODO】 再补充一些，尤其是brk。
   堆内存由glibc管理。每次调用brk改变堆内存大小。系统调用是有代价的，所以每次申请实际上会多申请一些。反正返回的也是虚拟地址，只有访问的时候，才会触发缺页中断而产生实际物理内存映射。
4. 缺页异常的信息由CPU提供，但是映射是MMU负责，CPU是如何知道这些信息的呢？


<!-- 阅读位置，电子书121/纸质书109页 -->

<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/21/  在线阅读 -->

<!-- https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt -->