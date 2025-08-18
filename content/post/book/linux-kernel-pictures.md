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

> 阅读期间，使用[linux kernel](https://kernel.org)，下载版本6.12.7

> 也推荐直接来这个网站看linux kernel：[bootlin](https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt)。也有很多其他重量级开源项目。

> 本书有很多细节，汇编代码，因此也建议作为科普，工具书阅读。有需要的时候可以回来看看。

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

1. 内存寻址
   
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

2. 物理内存的管理
    
    > 联动一下博客中的：[边学边用linux-内存管理]({{<relref "/content/post/book/linux-memory.md#Buddy">}})

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


3. 虚拟内存的管理
   
   每一个进程的线性地址空间（虚拟地址）划分分为内核态和用户态。内核态起始位置就是之前见到过的宏，```PAGE_OFFSET```。

   在x86时期，空间有限，内核虽然一般有1G的线性空间，但是并不能直接映射1G的物理内存。一般只能直接映射一部分（896M），剩下的部分保留满足其他需秋，直接映射的部分叫```Low Memory```,剩下的部分是```High Memory```。注意x86时期，物理内存是可以超过4G的，但是线性空间只有4G。

   所谓直接映射，就是映射后的虚拟地址和物理地址有直接关系，在前面的代码中也能看到：$va = vp + PAGEOFFSET$。此时映射状态（左侧物理内存，右侧线性空间）

   ![x86-va-pa](/images/book/linux-pic/x86-va-pa.png)

   到了x86-64时代，线性空间足够大了，不再区分```Low/High Memory```。

   内核线性空间（从高地址到低地址）内部还分为若干区域：
   - 32位：固定映射区、永久映射区、CPU Entry区、动态映射区、直接映射区
   - 64位：-

    ![x86-va-space](x86-va-space.png)
    ![x64-va-space](x64-va-space.png)
    > 64位可能有4、5级页表等不同情况，这里是5级的布局。

   内核空间实际上是进程间直接、或者间接共享的。可以理解为用户态空间互相独立，内核态空间共享。正因如此，用户空间的页表需要进程自行维护，是用户页表。而内核页表很多情况下是相同的，属于公共的部分。

   <!-- 阅读位置，电子书81/纸质书69页 -->

<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/21/  在线阅读 -->

## 课后问题
1. 进程控制块中包含了进程页表的基址，那进程控制块本身所在的虚拟内存，由谁的页表管理，以及其内存基址如何存储？如何避免套娃问题？
   
   涉及到内核启动、分页机制、进程管理的启动

1. low memory、high memory是什么