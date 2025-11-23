---
title: "图解Linux内核 读书笔记（下）"
date: 2025-10-22T20:43:15+08:00
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
本文是下半部分：进程管理和信号。
<!--more-->
# 进程
进程是一个非常复杂的话题，涵盖调度、通信等多方面。

## 数据结构
先来看进程的数据结构。内核定义了`task_struct`，这个结构体实在是太大了，这里给一下[链接](https://elixir.bootlin.com/linux/v6.2.16/source/include/linux/sched.h#L737)。然后摘出书上所点出的重要字段。

```c
struct task_struct {
	unsigned int			__state; // 进程状态
	void				*stack; // 进程内核栈
	int				recent_used_cpu;
	unsigned int			flags; // 进程标记
	
    // 优先级相关
	int				prio;
	int				static_prio;
	int				normal_prio;
	unsigned int			rt_priority;
    
    // 调度相关
	struct sched_entity		se;
	struct sched_rt_entity		rt;
	struct sched_dl_entity		dl;
	const struct sched_class	*sched_class;

	struct list_head		tasks; // 将当前结构体添加到进程链表
    
    // 进程的内存信息
	struct mm_struct		*mm;
	struct mm_struct		*active_mm;

    // 退出信息
	int				exit_state;
	int				exit_code;
	int				exit_signal;
	
    // 进程id、线程组id
    // 注意区分pid_t和pid结构体
	pid_t				pid;
	pid_t				tgid;

    // 父进程
	/* Real parent process: */
	struct task_struct __rcu	*real_parent;

	/* Recipient of SIGCHLD, wait4() reports: */
	struct task_struct __rcu	*parent;

    // 子进程链表头
	/*
	 * Children/sibling form the list of natural children:
	 */
	struct list_head		children;
    // 兄弟进程链表，这个链表头就是父进程的children字段值
	struct list_head		sibling;
    // 线程组的领导进程
	struct task_struct		*group_leader;

    // 进程对应的pid
	/* PID/PID hash table linkage. */
	struct pid			*thread_pid;
	struct hlist_node		pid_links[PIDTYPE_MAX];
    // 加入线程组链表
	struct list_head		thread_group;
	
	/* Objective and real subjective task credentials (COW): */
	const struct cred __rcu		*real_cred;

	/* Effective (overridable) subjective task credentials (COW): */
	const struct cred __rcu		*cred;

	/* Filesystem information: */
	struct fs_struct		*fs;

	/* Open file information: */
	struct files_struct		*files;

#ifdef CONFIG_IO_URING
	struct io_uring_task		*io_uring;
#endif

    // 管理进程的多种namespace
	/* Namespaces: */
	struct nsproxy			*nsproxy;

    // 信号处理相关
	/* Signal handlers: */
	struct signal_struct		*signal;
	struct sighand_struct __rcu		*sighand;
	sigset_t			blocked;
	sigset_t			real_blocked;
	/* Restored if set_restore_sigmask() was used: */
	sigset_t			saved_sigmask;
	struct sigpending		pending;
	
    // 平台有关信息
	/* CPU-specific state of this task: */
	struct thread_struct		thread;

	/*
	 * WARNING: on x86, 'thread_struct' contains a variable-sized
	 * structure.  It *MUST* be at the end of 'task_struct'.
	 *
	 * Do not put anything below here!
	 */
};
```

进程的状态`__state`包括很多种取值，代表进程当前的状态，如`TASK_DEAD`、`TASK_RUNNING`。而进程的标志，则类似于一种附加信息，代表进程当前的一些特性，比如空闲（`PF_IDLE`），运行在虚拟CPU上（`PF_VCPU`）。

内核栈和用户栈，分别在进程出于内核态和用户态的时候切换并使用。但内核栈有几个比较有趣的特点：
1. 内核栈一般比较小，几个KB，由THREAD_SIZE设置。
2. 每次进入内核态的时候，内核栈其实都是从起始位置重新开始使用的。这是因为内核信息并不需要保留，当内核处理完成，内核栈也就可以销毁了。

Linux的设计下，进程和线程没有做本质的区分，但是有些时候还是不能等价看待的（一定程度上和POSIX标准要求有关）。线程就是轻量级进程。而线程组、进程组、会话，都是指一组进程的集合。他们之间的顺序是：会话可以有多个进程组，进程组可以有多个线程组。一个用户登录就会开启一个会话。使用`ps`命令时，可以看到的分别是：PID进程id、PPID父进程id、PGID进程组id、SID会话id、LWP线程id、NLWP线程组的线程数。

进程的查找是通过pid结构体辅助完成的。用户使用的进程号pid_t需要最终找到task_struct才行。
```c
struct pid
{
	refcount_t count;
	unsigned int level;
	spinlock_t lock;
    // 对应四种链表的头：进程、线程组、进程组、会话，这是四种不同的PID TYPE
	/* lists of tasks that use this pid */
	struct hlist_head tasks[PIDTYPE_MAX];
	struct hlist_head inodes;
	/* wait queue for pidfd notifications */
	wait_queue_head_t wait_pidfd;
	struct rcu_head rcu;
	struct upid numbers[1];
};

struct upid {
    // 进程id
	int nr;
    // 进程所属的namespace的信息
	struct pid_namespace *ns;
};

struct pid_namespace {
	struct idr idr;
	struct rcu_head rcu;
	unsigned int pid_allocated;
	struct task_struct *child_reaper;
	struct kmem_cache *pid_cachep;
	unsigned int level;
	struct pid_namespace *parent;
#ifdef CONFIG_BSD_PROCESS_ACCT
	struct fs_pin *bacct;
#endif
	struct user_namespace *user_ns;
	struct ucounts *ucounts;
	int reboot;	/* group exit code if this pidns was rebooted */
	struct ns_common ns;
} __randomize_layout;


```
查找过程是：使用进程id在pid_namespace中查找，得到upid，再通过upid找到pid（使用container_of）。从pid就可以利用hlist_找到task_struct了。

进程间的亲属关系也很重要：
1. 亲属关系（父子、兄弟）。其中父进程并不是一直不变的，父进程退出，或者进程trace，`parent`会被改编。`real_parent`字段则始终是创建的父进程
2. 所属关系：线程组、进程组、会话的关系。

![task_struct_links](task_struct_links.png)

## 进程创建

<!-- 进度 电子书209 /纸质197 -->

<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/202/  在线阅读 -->

<!-- https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt -->

<!-- https://elixir.bootlin.com/linux/v6.2.16/source -->

<!-- 本文为第二部分，即电子书201页之后，纸质书189页之后。 -->