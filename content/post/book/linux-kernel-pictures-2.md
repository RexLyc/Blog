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
先来看进程的数据结构。内核定义了`task_struct`，这个结构体实在是太大了，这里给一下[链接](https://elixir.bootlin.com/linux/v6.2.16/source/include/linux/sched.h#L737)。然后在下面摘出书上所点出的重要字段。

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
    // 链接到四种目标进程类型pid中，进程、线程组、进程组、会话
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

进程的状态`__state`包括很多种取值，代表进程当前的状态，如`TASK_DEAD`、`TASK_RUNNING`。而进程的标志`flags`，则类似于一种附加信息，代表进程当前的一些特性，比如空闲（`PF_IDLE`），运行在虚拟CPU上（`PF_VCPU`）。

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
    // 该值等于进程id
	int nr;
    // 进程所属的namespace的信息
	struct pid_namespace *ns;
};

struct pid_namespace {
    // 维护id和upid的关系
	struct idr idr;
	struct rcu_head rcu;
	unsigned int pid_allocated;
    // 指向该pid_namespace的init进程，用于作为托孤进程
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
查找过程是：使用进程id即可在pid_namespace中，使用idr查找得到upid，再通过upid找到pid（使用container_of） 。从pid就可以利用hlist_找到task_struct了。

> 查找过程中，`idr`字段起到了很大的作用。`idr`其实是基数树（radix tree）的数据结构。所以实际上是id + r的缩写。注意xarray机制已经开始替换idr。idr机制你可以简单理解为，我们能够使用一个id，来查找一个数据结构的指针。idr的更多资料，可以参考[基数树（Radix Tree）](https://juejin.cn/post/6933244263241089037)，内核文档[idr和ida](https://docs.kernel.org/translations/zh_CN/core-api/idr.html)。

进程间的亲属关系也很重要：
1. 亲属关系（父子、兄弟）。其中父进程并不是一直不变的，父进程退出，或者进程trace，`parent`会被改编。`real_parent`字段则始终是创建的父进程
2. 所属关系：线程组、进程组、会话的关系。
3. task_struct结构体中的`pid_links`，以及pid结构体中的`tasks`，是将同类型的进程链接到一起的链表，如下图所示。属于同一个PIDTYPE_TGID线程组、PIDTYPE_PGID进程组、PIDTYPE_SID会话的进程，会被连接到同一个链表中。

![task_struct_links](/images/book/linux-pic/task_struct_links.png)

## 进程创建

进程创建最终是通过`kernel_clone`函数实现。其中复杂的部分是`copy_process`。

```c
/*
 * This creates a new process as a copy of the old one,
 * but does not actually start it yet.
 *
 * It copies the registers, and all the appropriate
 * parts of the process environment (as per the clone
 * flags). The actual kick-off is left to the caller.
 */
static __latent_entropy struct task_struct *copy_process(
					struct pid *pid,
					int trace,
					int node,
					struct kernel_clone_args *args)
{
	int pidfd = -1, retval;
	struct task_struct *p;
	struct multiprocess_signals delayed;
	struct file *pidfile = NULL;
	const u64 clone_flags = args->flags;
	struct nsproxy *nsp = current->nsproxy;

	/*
	 * Don't allow sharing the root directory with processes in a different
	 * namespace
	 */
	if ((clone_flags & (CLONE_NEWNS|CLONE_FS)) == (CLONE_NEWNS|CLONE_FS))
		return ERR_PTR(-EINVAL);

	if ((clone_flags & (CLONE_NEWUSER|CLONE_FS)) == (CLONE_NEWUSER|CLONE_FS))
		return ERR_PTR(-EINVAL);
    
    // 此处省略大量类似检查
    // ...

	/*
     * 信号处理检查点，前面的现在处理，以后的都会触发到多个进程了
	 * Force any signals received before this point to be delivered
	 * before the fork happens.  Collect up signals sent to multiple
	 * processes that happen during the fork and delay them so that
	 * they appear to happen after the fork.
	 */
	sigemptyset(&delayed.signal);
	INIT_HLIST_NODE(&delayed.node);

	spin_lock_irq(&current->sighand->siglock);
	if (!(clone_flags & CLONE_THREAD))
		hlist_add_head(&delayed.node, &current->signal->multiprocess);
	recalc_sigpending();
	spin_unlock_irq(&current->sighand->siglock);
	retval = -ERESTARTNOINTR;
	if (task_sigpending(current))
		goto fork_out;

	retval = -ENOMEM;
    // 重点1：申请内存，创建新进程的task struct
	p = dup_task_struct(current, node);
	if (!p)
		goto fork_out;
	
    // 省略一些flag处理
    // ...

    // 重点2：为task struct中的cred字段赋值
    // cred字段涉及到进程安全相关的内容：uid、gid、capabilities等
    // 根据clone参数，cred所使用的值可能是继承自当前进程的，也可能需要创建新的
	retval = copy_creds(p, clone_flags);
	if (retval < 0)
		goto bad_fork_free;

    // 省略一些初始化和拷贝
    // ...

    // 设置时间：utime、utimescaled、stime、stimescaled、
    //          gtime、start_time、real_start_time等
    // 此处主要是清0
	p->utime = p->stime = p->gtime = 0;
#ifdef CONFIG_ARCH_HAS_SCALED_CPUTIME
	p->utimescaled = p->stimescaled = 0;
#endif
	prev_cputime_init(&p->prev_cputime);

#ifdef CONFIG_VIRT_CPU_ACCOUNTING_GEN
	seqcount_init(&p->vtime.seqcount);
	p->vtime.starttime = 0;
	p->vtime.state = VTIME_INACTIVE;
#endif

    // 又有一些初始化，NUMA，cpuset
    // ...

	/* Perform scheduler related setup. Assign this task to a CPU. */
    // 重点3：和进程调度有关的设置，初始化task struct中的se、dl、rt等
    //       调整优先级、调度策略
	retval = sched_fork(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_policy;

    // 大量copy_*，包括fs、file、sighand、signal、mm等
    // ...省略一些copy

    // semundo和进程间通信有关
    // 其实所有的copy执行的逻辑都差不多，要么和当前进程共享，要么新建一个
	retval = copy_semundo(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_security;
    // 不再open的文件不会共享
	retval = copy_files(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_semundo;
	retval = copy_fs(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_files;
	retval = copy_sighand(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_fs;
	retval = copy_signal(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_sighand;
    // mm 这里下面展开
	retval = copy_mm(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_signal;
    // 是否需要新ns，创建新ns需要admin权限
    // 创建ns也是一个一个ns判断的，其中部分ns可以共享
    // 这里可以再回上面看一下pid_namespace，这个比较重要
	retval = copy_namespaces(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_mm;
    // io_context
	retval = copy_io(clone_flags, p);
	if (retval)
		goto bad_fork_cleanup_namespaces;
    // 平台相关，配置新进程的状态，thread_struct结构体
    // 比如保存SP寄存器所需要的值（栈的起点），存储执行起点等
    // 像在x86平台上，childregs->ax配置的就是fork的返回值
	retval = copy_thread(p, args);
	if (retval)
		goto bad_fork_cleanup_io;

	stackleak_task_init(p);

    // 在指定ns中，申请pid
    // 注意一个进程需要在自己的pid namespace，以及所有父、祖父，一直到初始init_pid_ns中，都获得一个id。
	if (pid != &init_struct_pid) {
		pid = alloc_pid(p->nsproxy->pid_ns_for_children, args->set_tid,
				args->set_tid_size);
		if (IS_ERR(pid)) {
			retval = PTR_ERR(pid);
			goto bad_fork_cleanup_thread;
		}
	}

    // 省略一些检查和配置
    // ...

    // 接下来有一些赋值，并建立新进程和其他进程间的关系

    // 简单来说，关系由新进程和创建进程共同决定
    // 新进程可能是一个线程，此时它和创建进程同属一个线程组，因此要设置相同的线程组信息
    // 同理，创建进程可能是一个线程，也就是说其实可能由4种情况：
    // 线程/进程 fork出来 线程/进程
    // 赋值的主要内容，就是看是否创建新的线程组，以及是否加入相关的进程链表。

	/* CLONE_PARENT re-uses the old parent */
	if (clone_flags & (CLONE_PARENT|CLONE_THREAD)) {
        // 进入到这个分支，新父进程是当前进程的父进程
        // CLONE_PARENT选项，代表将父进程标记也保留了下来了
        // CLONE_THREAD选项，代表创建的是线程，而非进程
        // 其实这两个选项的功能类似，新创建的进程和当前进程其实是兄弟关系，或者是当前线程组下的一个线程
		p->real_parent = current->real_parent;
		p->parent_exec_id = current->parent_exec_id;
        // exit_signal会用于判断是否是线程组领导
		if (clone_flags & CLONE_THREAD)
			p->exit_signal = -1;
		else
			p->exit_signal = current->group_leader->exit_signal;
	} else {
        // 否则，进入这个分支，新进程的父进程就是当前进程
		p->real_parent = current;
		p->parent_exec_id = current->self_exec_id;
		p->exit_signal = args->exit_signal;
	}

    // 省略一些过程
    // ...

	if (likely(p->pid)) {
        // 设置进程的parent字段
		ptrace_init_task(p, (clone_flags & CLONE_PTRACE) || trace);

		init_task_pid(p, PIDTYPE_PID, pid);
        //
		if (thread_group_leader(p)) { // 领导进程，需要链接到父进程的链表：线程组、进程组、会话组的链表中
			init_task_pid(p, PIDTYPE_TGID, pid);
			init_task_pid(p, PIDTYPE_PGID, task_pgrp(current));
			init_task_pid(p, PIDTYPE_SID, task_session(current));

			if (is_child_reaper(pid)) {
				ns_of_pid(pid)->child_reaper = p;
				p->signal->flags |= SIGNAL_UNKILLABLE;
			}
			p->signal->shared_pending.signal = delayed.signal;
			p->signal->tty = tty_kref_get(current->signal->tty);
			/*
			 * Inherit has_child_subreaper flag under the same
			 * tasklist_lock with adding child to the process tree
			 * for propagate_has_child_subreaper optimization.
			 */
			p->signal->has_child_subreaper = p->real_parent->signal->has_child_subreaper ||
							 p->real_parent->signal->is_child_subreaper;
			list_add_tail(&p->sibling, &p->real_parent->children);
			list_add_tail_rcu(&p->tasks, &init_task.tasks);
			attach_pid(p, PIDTYPE_TGID);
			attach_pid(p, PIDTYPE_PGID);
			attach_pid(p, PIDTYPE_SID);
			__this_cpu_inc(process_counts);
		} else {
			current->signal->nr_threads++;
			current->signal->quick_threads++;
			atomic_inc(&current->signal->live);
			refcount_inc(&current->signal->sigcnt);
			task_join_group_stop(p);
			list_add_tail_rcu(&p->thread_group,
					  &p->group_leader->thread_group);
			list_add_tail_rcu(&p->thread_node,
					  &p->signal->thread_head);
		}
        // 将p链接到pid的链表中，建立新进程和pid的关系
		attach_pid(p, PIDTYPE_PID);
		nr_threads++;
	}
	
    // 省略剩余过程、跳转标签、错误返回
    // ...
}
```

阅读上方代码可以知道，在创建过程中，其实就是通过标志位来区分行为，尤其是区分创建的是进程，还是线程。

而其中`copy_mm`比较复杂。在下方单独展开。

### 复制mm

#### copy_mm

task_struct中的mm和active_mm字段和内存管理有关。指向mm_struct结构体指针。

- mm：表示进程管理的内存信息。mm管理的内存，至少有一部分是属于进程本身的。
- active_mm：表示当前进程所使用的内存信息。active_mm使用的内存，可能不属于进程。

回顾一下mm_struct的内容
```c
// 依旧是截取部分
struct mm_struct {
	struct {
        // 进程的vma组成的链表头
		struct maple_tree mm_mt;

        // 指向pgd
        pgd_t * pgd;

		/**
		 * @mm_users: The number of users including userspace.
		 *
		 * Use mmget()/mmget_not_zero()/mmput() to modify. When this
		 * drops to 0 (i.e. when the task exits and there are no other
		 * temporary reference holders), we also release a reference on
		 * @mm_count (which may then free the &struct mm_struct if
		 * @mm_count also drops to 0).
		 */
        //  引用计数
		atomic_t mm_users;

		/**
		 * @mm_count: The number of references to &struct mm_struct
		 * (@mm_users count as 1).
		 *
		 * Use mmgrab()/mmdrop() to modify. When this drops to 0, the
		 * &struct mm_struct is freed.
		 */
        //  引用计数
		atomic_t mm_count;

		/* Architecture-specific MM context */
		mm_context_t context;

		/* store ref to file /proc/<pid>/exe symlink points to */
        // 进程执行的文件，可以为null
		struct file __rcu *exe_file;

	} __randomize_layout;

	/*
	 * The mm_cpumask needs to be at the end of mm_struct, because it
	 * is dynamically sized based on nr_cpu_ids.
	 */
	unsigned long cpu_bitmap[];
};
```

其中mm_user和mm_count
1. 分别代表引用的线程数，和引用的线程组的数量。一个线程组的mm_users清为0，只会为mm_count减少1。
2. 进程之间可以借用内存。借用时就会导致mm_count加一。当某一个进程退出时，或者不再使用内存。就可以释放一部分。

释放的逻辑主要位于mmput。释放是递进的。mm_user为0时，可以释放aio、mmap、exe_file。mm_count为0时，释放pgd、context和mm_struct本身。
```c
/*
 * Decrement the use count and release all resources for an mm.
 */
void mmput(struct mm_struct *mm)
{
	might_sleep();

	if (atomic_dec_and_test(&mm->mm_users))
		__mmput(mm);
}

static inline void __mmput(struct mm_struct *mm)
{
	VM_BUG_ON(atomic_read(&mm->mm_users));

	uprobe_clear_state(mm);
	exit_aio(mm);
	ksm_exit(mm);
	khugepaged_exit(mm); /* must run before exit_mmap */
	exit_mmap(mm);
	mm_put_huge_zero_page(mm);
	set_mm_exe_file(mm, NULL);
	if (!list_empty(&mm->mmlist)) {
		spin_lock(&mmlist_lock);
		list_del(&mm->mmlist);
		spin_unlock(&mmlist_lock);
	}
	if (mm->binfmt)
		module_put(mm->binfmt->module);
	lru_gen_del_mm(mm);
	mmdrop(mm);
}

static inline void mmdrop(struct mm_struct *mm)
{
	/*
	 * The implicit full barrier implied by atomic_dec_and_test() is
	 * required by the membarrier system call before returning to
	 * user-space, after storing to rq->curr.
	 */
	if (unlikely(atomic_dec_and_test(&mm->mm_count)))
		__mmdrop(mm);
}


/*
 * Called when the last reference to the mm
 * is dropped: either by a lazy thread or by
 * mmput. Free the page directory and the mm.
 */
void __mmdrop(struct mm_struct *mm)
{
	int i;

	BUG_ON(mm == &init_mm);
	WARN_ON_ONCE(mm == current->mm);
	WARN_ON_ONCE(mm == current->active_mm);
	mm_free_pgd(mm);
	destroy_context(mm);
	mmu_notifier_subscriptions_destroy(mm);
	check_mm(mm);
	put_user_ns(mm->user_ns);
	mm_pasid_drop(mm);

	for (i = 0; i < NR_MM_COUNTERS; i++)
		percpu_counter_destroy(&mm->rss_stat[i]);
	free_mm(mm);
}

```

了解了内存借用，回头来看copy_mm流程

```c
static int copy_mm(unsigned long clone_flags, struct task_struct *tsk)
{
	struct mm_struct *mm, *oldmm;

	tsk->min_flt = tsk->maj_flt = 0;
	tsk->nvcsw = tsk->nivcsw = 0;
#ifdef CONFIG_DETECT_HUNG_TASK
	tsk->last_switch_count = tsk->nvcsw + tsk->nivcsw;
	tsk->last_switch_time = 0;
#endif

	tsk->mm = NULL;
	tsk->active_mm = NULL;

	/*
	 * Are we cloning a kernel thread?
	 *
	 * We need to steal a active VM for that..
	 */
	oldmm = current->mm;
    // 内核线程是没有自己管理的内存的，因此直接返回
	if (!oldmm)
		return 0;

	if (clone_flags & CLONE_VM) {
		mmget(oldmm);
		mm = oldmm;
	} else {
        // 新建一个mm_struct，并复制当前进程的mm_struct的值
        // 在下面展开
		mm = dup_mm(tsk, current->mm);
		if (!mm)
			return -ENOMEM;
	}

	tsk->mm = mm;
	tsk->active_mm = mm;
	return 0;
}

/**
 * dup_mm() - duplicates an existing mm structure
 * @tsk: the task_struct with which the new mm will be associated.
 * @oldmm: the mm to duplicate.
 *
 * Allocates a new mm structure and duplicates the provided @oldmm structure
 * content into it.
 *
 * Return: the duplicated mm or NULL on failure.
 */
static struct mm_struct *dup_mm(struct task_struct *tsk,
				struct mm_struct *oldmm)
{
	struct mm_struct *mm;
	int err;

    // 申请新的mm_struct
	mm = allocate_mm();
	if (!mm)
		goto fail_nomem;

    // 复制
	memcpy(mm, oldmm, sizeof(*mm));

    // 初始化一些字段，在下面展开
	if (!mm_init(mm, tsk, mm->user_ns))
		goto fail_nomem;
    
    // 同理，初始化一些字段
	err = dup_mmap(mm, oldmm);
	if (err)
		goto free_pt;

	mm->hiwater_rss = get_mm_rss(mm);
	mm->hiwater_vm = mm->total_vm;

	if (mm->binfmt && !try_module_get(mm->binfmt->module))
		goto free_pt;

	return mm;

free_pt:
	/* don't put binfmt in mmput, we haven't got module yet */
	mm->binfmt = NULL;
	mm_init_owner(mm, NULL);
	mmput(mm);

fail_nomem:
	return NULL;
}
```


#### mm_init

```c
static struct mm_struct *mm_init(struct mm_struct *mm, struct task_struct *p,
	struct user_namespace *user_ns)
{
	int i;
    // 省略一些mm_struct字段的初始化
    // ...

    // 初始化进程的 pgd
    // 回忆一下现在普遍使用的五级页表，PGD、P4D、PUD、PMD、PT
    // 强烈建议复习内存章节哦！
	if (mm_alloc_pgd(mm))
		goto fail_nopgd;

	if (init_new_context(p, mm))
		goto fail_nocontext;

	for (i = 0; i < NR_MM_COUNTERS; i++)
		if (percpu_counter_init(&mm->rss_stat[i], 0, GFP_KERNEL_ACCOUNT))
			goto fail_pcpu;

	mm->user_ns = get_user_ns(user_ns);
	lru_gen_init_mm(mm);
	return mm;

fail_pcpu:
	while (i > 0)
		percpu_counter_destroy(&mm->rss_stat[--i]);
	destroy_context(mm);
fail_nocontext:
	mm_free_pgd(mm);
fail_nopgd:
	free_mm(mm);
	return NULL;
}

static inline int mm_alloc_pgd(struct mm_struct *mm)
{
	mm->pgd = pgd_alloc(mm);
	if (unlikely(!mm->pgd))
		return -ENOMEM;
	return 0;
}

// 注意pgd_alloc是平台相关的，下面是x86的实现
pgd_t *pgd_alloc(struct mm_struct *mm)
{
	pgd_t *pgd;
	pmd_t *u_pmds[MAX_PREALLOCATED_USER_PMDS];
	pmd_t *pmds[MAX_PREALLOCATED_PMDS];

	pgd = _pgd_alloc();

	if (pgd == NULL)
		goto out;

	mm->pgd = pgd;

    // 举例1：为什么是平台相关
    // 仅在x86（即32位），且使能PAE的情况下，预申请PMD
    // PAE时，三级页表位数 2 + 9 + 9。PGD、PMD、PT
    // 此时只需4页内存(2^2)，就可以存下所有的PMD
    // 其实就是此时PMD总量不多，提前分配可以接受
    // 如果PMD总量很多，肯定没必要这么做了
	if (sizeof(pmds) != 0 &&
			preallocate_pmds(mm, pmds, PREALLOCATED_PMDS) != 0)
		goto out_free_pgd;

	if (sizeof(u_pmds) != 0 &&
			preallocate_pmds(mm, u_pmds, PREALLOCATED_USER_PMDS) != 0)
		goto out_free_pmds;

	if (paravirt_pgd_alloc(mm) != 0)
		goto out_free_user_pmds;

	/*
	 * Make sure that pre-populating the pmds is atomic with
	 * respect to anything walking the pgd_list, so that they
	 * never see a partially populated pgd.
	 */
	spin_lock(&pgd_lock);

    // 重点：复制内核对应的pgd项
    // 这一步的重点是，内核空间是高位地址，因此只有后面的部分才是内核空间
    // 而不同的系统位数，计算方式略有区别
	pgd_ctor(mm, pgd);


    // 将前面申请得到的pmd和pgd进行关联，将pgd指向对应pmd
	if (sizeof(pmds) != 0)
		pgd_prepopulate_pmd(mm, pgd, pmds);

	if (sizeof(u_pmds) != 0)
		pgd_prepopulate_user_pmd(mm, pgd, u_pmds);

	spin_unlock(&pgd_lock);

	return pgd;

out_free_user_pmds:
	if (sizeof(u_pmds) != 0)
		free_pmds(mm, u_pmds, PREALLOCATED_USER_PMDS);
out_free_pmds:
	if (sizeof(pmds) != 0)
		free_pmds(mm, pmds, PREALLOCATED_PMDS);
out_free_pgd:
	_pgd_free(pgd);
out:
	return NULL;
}

```

> 使能PAE其实已经不再重要了，现在普遍都是x86_64机器。但是这其中带来的，关于多级页表、以及对页表项的理解很重要。

使能PAE，物理地址扩展（Physical Address Extension，简称 PAE）。此时页表项变为64位一个（每一级的页表项都是64位），而非32位一个。在4KB分页情况下，原本一个页中，可以存放1024个页表项，现在只能存512个。但每个页表项的物理页框号，从原来的20位，扩充为最多52位。不过仍受制于CPU实际支持的物理内存大小。**Intel规定，使能PAE至少支持36位物理地址**，因此一般以36位为准。此时虽然系统仍运行在32位，但物理内存可以达到64GB。

另一点，32位情况下，一般默认两级页表，也就是10 + 10。那为什么使能PAE之后，各级页表位数要变为 2 + 9 + 9呢？其实这始终是在配合页的大小4KB。因为页表项变大。所以一个页只能存512个页表项。也就是每一级的索引最多只需要9位。此时就肯定要修改页表的级数了。

另外一个**最重要的事情**，就是理解对内核PGD的复制。`pgd_ctor`内部，会从swappger_pg_dir复制内核项。linux的内核部分是跨进程共享的。为了完成这个共享，就需要共享或者复制内核部分的PGD。并配合惰性修改（内核的PGD大部分静态，但也有动态,比如vmalloc）。具体共享还是复制，受选项的控制，而且使能PAE还会让这一步骤更复杂（和PMD的预分配有关，不仅要共享PGD，还需要拷贝PMD）。

![pgd and swapper_pg_dir](/images/book/linux-pic/pgd_ctor.png)

在这种设计下，如果出现内核内存动态变化，而进程PGD未能更新的情况，此时就会触发缺页中断，并由vmalloc_fault函数完成页的同步。

而到了x86_64，swapper_pg_dir的处理又有不同。因为64位的地址空间相当之大。没有必要扣扣嗖嗖了。PGD中和vmalloc有关的项直接填上固定的值了。此时内核PGD几乎是冻结不变的，

![x86_64 preallocate_vmalloc_pages优化](/images/book/linux-pic/prealloc_vmalloc_pages.png)

#### dup_mmap

上一节我们谈到了内核PGD的共享。当前进程的内存映射也是要共享的啦。这部分工作由`dup_mmap`完成。

```c
static __latent_entropy int dup_mmap(struct mm_struct *mm,
					struct mm_struct *oldmm)
{
	struct vm_area_struct *mpnt, *tmp;
	int retval;
	unsigned long charge = 0;
	LIST_HEAD(uf);
	MA_STATE(old_mas, &oldmm->mm_mt, 0, 0);
	MA_STATE(mas, &mm->mm_mt, 0, 0);

	uprobe_start_dup_mmap();
	if (mmap_write_lock_killable(oldmm)) {
		retval = -EINTR;
		goto fail_uprobe_end;
	}
	flush_cache_dup_mm(oldmm);
	uprobe_dup_mmap(oldmm, mm);
	/*
	 * Not linked in yet - no deadlock potential:
	 */
	mmap_write_lock_nested(mm, SINGLE_DEPTH_NESTING);

	/* No ordering required: file already has been exposed. */
	dup_mm_exe_file(mm, oldmm);

	mm->total_vm = oldmm->total_vm;
	mm->data_vm = oldmm->data_vm;
	mm->exec_vm = oldmm->exec_vm;
	mm->stack_vm = oldmm->stack_vm;

	retval = ksm_fork(mm, oldmm);
	if (retval)
		goto out;
	khugepaged_fork(mm, oldmm);

	retval = mas_expected_entries(&mas, oldmm->map_count);
	if (retval)
		goto out;

	mt_clear_in_rcu(mas.tree);
	mas_for_each(&old_mas, mpnt, ULONG_MAX) {
		struct file *file;

		if (mpnt->vm_flags & VM_DONTCOPY) {
			vm_stat_account(mm, mpnt->vm_flags, -vma_pages(mpnt));
			continue;
		}
		charge = 0;
		/*
		 * Don't duplicate many vmas if we've been oom-killed (for
		 * example)
		 */
		if (fatal_signal_pending(current)) {
			retval = -EINTR;
			goto loop_out;
		}
		if (mpnt->vm_flags & VM_ACCOUNT) {
			unsigned long len = vma_pages(mpnt);

			if (security_vm_enough_memory_mm(oldmm, len)) /* sic */
				goto fail_nomem;
			charge = len;
		}
        // 内存共享只需要共享需要的部分，也就是遍历mm_struct中的mm_mt字段
        // 该字段是vm_area_struct对象组成的maple_tree，拷贝需要的vma即可
        // 申请新的vma
		tmp = vm_area_dup(mpnt);
		if (!tmp)
			goto fail_nomem;
		retval = vma_dup_policy(mpnt, tmp);
		if (retval)
			goto fail_nomem_policy;
		tmp->vm_mm = mm;
		retval = dup_userfaultfd(tmp, &uf);
		if (retval)
			goto fail_nomem_anon_vma_fork;
		if (tmp->vm_flags & VM_WIPEONFORK) {
			/*
			 * VM_WIPEONFORK gets a clean slate in the child.
			 * Don't prepare anon_vma until fault since we don't
			 * copy page for current vma.
			 */
			tmp->anon_vma = NULL;
		} else if (anon_vma_fork(tmp, mpnt))
			goto fail_nomem_anon_vma_fork;
		tmp->vm_flags &= ~(VM_LOCKED | VM_LOCKONFAULT);
		file = tmp->vm_file;
		if (file) {
			struct address_space *mapping = file->f_mapping;

			get_file(file);
			i_mmap_lock_write(mapping);
			if (tmp->vm_flags & VM_SHARED)
				mapping_allow_writable(mapping);
			flush_dcache_mmap_lock(mapping);
			/* insert tmp into the share list, just after mpnt */
			vma_interval_tree_insert_after(tmp, mpnt,
					&mapping->i_mmap);
			flush_dcache_mmap_unlock(mapping);
			i_mmap_unlock_write(mapping);
		}

		/*
		 * Copy/update hugetlb private vma information.
		 */
		if (is_vm_hugetlb_page(tmp))
			hugetlb_dup_vma_private(tmp);

		/* Link the vma into the MT */
		mas.index = tmp->vm_start;
		mas.last = tmp->vm_end - 1;
        // 插入新进程的maple_tree
		mas_store(&mas, tmp);
		if (mas_is_err(&mas))
			goto fail_nomem_mas_store;

		mm->map_count++;
        // 拷贝对应vma涉及的各级页表
		if (!(tmp->vm_flags & VM_WIPEONFORK))
			retval = copy_page_range(tmp, mpnt);

		if (tmp->vm_ops && tmp->vm_ops->open)
			tmp->vm_ops->open(tmp);

		if (retval)
			goto loop_out;
	}
	/* a new mm has just been created */
	retval = arch_dup_mmap(oldmm, mm);
loop_out:
	mas_destroy(&mas);
	if (!retval)
		mt_set_in_rcu(mas.tree);
out:
	mmap_write_unlock(mm);
	flush_tlb_mm(oldmm);
	mmap_write_unlock(oldmm);
	dup_userfaultfd_complete(&uf);
fail_uprobe_end:
	uprobe_end_dup_mmap();
	return retval;

fail_nomem_mas_store:
	unlink_anon_vmas(tmp);
fail_nomem_anon_vma_fork:
	mpol_put(vma_policy(tmp));
fail_nomem_policy:
	vm_area_free(tmp);
fail_nomem:
	retval = -ENOMEM;
	vm_unacct_memory(charge);
	goto loop_out;
}
```

创建进程时，对于用户空间下的内存共享。是一个很有价值的优化点。目前Linux是以vma为视角对各级页表进行拷贝。只处理那些需要共享的vma。但是需要完整拷贝各级页表，并在页表项上添加一些标记，用于控制COW。另外VMA中也有一些控制访存属性的内容。可以再复习一下内存章节。尤其是[虚拟内存]({{<relref "/content/post/book/linux-kernel-pictures-1.md#虚拟内存的管理">}})部分。

### 正式创建
回到整体视角。创建进程，其实最外层就是我们熟悉的`fork/vfork`系统调用。其内部就是调用`kernel_clone`来实现。还提供了一些参数可以控制一些行为，比如新进程和当前进程的运行顺序、是否共享`mm_struct`等。
- fork调用，不共享mm_struct，不保证执行顺序。
- vfork调用，共享mm_struct，保证新进程先执行（比当前进程先调度）。

![CLONE_VM](/images/book/linux-pic/clone_vm_flag.png)

子进程调度执行后，执行的起点是`ret_from_fork`。这也是一个硬件相关的实现。

![fork](fork_ret_twice.png)


但是fork只能用来创建进程。clone系统调用提供更多的参数，可以创建线程，底层还是`kernel_clone`。此时POSIX标准在这里就显得很重要，我们不需要关心Linux是如何看待线程的，只需要知道`pthread_create`即可。不过`pthread_create`的流程和`fork`之类的就会稍有区别。
- 第一点：pthread_create内部会指定CLONE_VM和stackaddr，也就是新线程虽然共享mm_struct，但是有自己的栈。

![thread stack](/images/book/linux-pic/thread_stack.png)

- 另一点：和fork的ret_from_fork不同，对于新线程来说，pthread_create的返回是需要从用户指定的start_thread开始。而这个入口是由glibc维护的`__clone`函数来处理，而不是内核。内核仍然是从ret_from_fork开始返回，然后会执行glibc提供的`__clone`函数的这段逻辑。其内部就是通过将寄存器、或者入栈了的寄存器值，保存pthread_create返回位置，或者用户要执行的函数位置。


### 内核线程
内核线程是一种只存在于内核态的进程。不会在用户态运行，多是内核中的服务进程，不需要属于自己的内存，因此`task_struct`中的mm字段为NULL。其flags字段的`PF_KTHREAD`标志位被置位。

内核线程必须由内核线程创建。但是内核线程可以转变为普通进程，只需要执行`do_execve`即可。

内核提供了一个单独的线程`kthreadd`来创建其他内核线程。其他线程的内核线程创建申请，都会添加到链表`kthread_create_list`中，并由kthreadd来遍历创建。

### 第一个进程
第一个进程是idle进程，不是动态创建的，基本上是固定在系统中的init_task变量。

```c
/*
 * Set up the first task table, touch at your own risk!. Base=0,
 * limit=0x1fffff (=2MB)
 */
struct task_struct init_task
#ifdef CONFIG_ARCH_TASK_STRUCT_ON_STACK
	__init_task_data
#endif
	__aligned(L1_CACHE_BYTES)
= {
#ifdef CONFIG_THREAD_INFO_IN_TASK
	.thread_info	= INIT_THREAD_INFO(init_task),
	.stack_refcount	= REFCOUNT_INIT(1),
#endif
	.__state	= 0,
	.stack		= init_stack,
	.usage		= REFCOUNT_INIT(2),
	.flags		= PF_KTHREAD,
	.prio		= MAX_PRIO - 20,
	.static_prio	= MAX_PRIO - 20,
	.normal_prio	= MAX_PRIO - 20,
	.policy		= SCHED_NORMAL,
	.cpus_ptr	= &init_task.cpus_mask,
	.user_cpus_ptr	= NULL,
	.cpus_mask	= CPU_MASK_ALL,
	.nr_cpus_allowed= NR_CPUS,
	.mm		= NULL,
	.active_mm	= &init_mm,
	.restart_block	= {
		.fn = do_no_restart_syscall,
	},
	.se		= {
		.group_node 	= LIST_HEAD_INIT(init_task.se.group_node),
	},
	.rt		= {
		.run_list	= LIST_HEAD_INIT(init_task.rt.run_list),
		.time_slice	= RR_TIMESLICE,
	},
	.tasks		= LIST_HEAD_INIT(init_task.tasks),
#ifdef CONFIG_SMP
	.pushable_tasks	= PLIST_NODE_INIT(init_task.pushable_tasks, MAX_PRIO),
#endif
#ifdef CONFIG_CGROUP_SCHED
	.sched_task_group = &root_task_group,
#endif
	.ptraced	= LIST_HEAD_INIT(init_task.ptraced),
	.ptrace_entry	= LIST_HEAD_INIT(init_task.ptrace_entry),
	.real_parent	= &init_task,
	.parent		= &init_task,
	.children	= LIST_HEAD_INIT(init_task.children),
	.sibling	= LIST_HEAD_INIT(init_task.sibling),
	.group_leader	= &init_task,
	RCU_POINTER_INITIALIZER(real_cred, &init_cred),
	RCU_POINTER_INITIALIZER(cred, &init_cred),
	.comm		= INIT_TASK_COMM,
	.thread		= INIT_THREAD,
	.fs		= &init_fs,
	.files		= &init_files,
#ifdef CONFIG_IO_URING
	.io_uring	= NULL,
#endif
	.signal		= &init_signals,
	.sighand	= &init_sighand,
	.nsproxy	= &init_nsproxy,
	.pending	= {
		.list = LIST_HEAD_INIT(init_task.pending.list),
		.signal = {{0}}
	},
	.blocked	= {{0}},
	.alloc_lock	= __SPIN_LOCK_UNLOCKED(init_task.alloc_lock),
	.journal_info	= NULL,
	INIT_CPU_TIMERS(init_task)
	.pi_lock	= __RAW_SPIN_LOCK_UNLOCKED(init_task.pi_lock),
	.timer_slack_ns = 50000, /* 50 usec default slack */
	.thread_pid	= &init_struct_pid,
	.thread_group	= LIST_HEAD_INIT(init_task.thread_group),
	.thread_node	= LIST_HEAD_INIT(init_signals.thread_head),
#ifdef CONFIG_AUDIT
	.loginuid	= INVALID_UID,
	.sessionid	= AUDIT_SID_UNSET,
#endif
#ifdef CONFIG_PERF_EVENTS
	.perf_event_mutex = __MUTEX_INITIALIZER(init_task.perf_event_mutex),
	.perf_event_list = LIST_HEAD_INIT(init_task.perf_event_list),
#endif
#ifdef CONFIG_PREEMPT_RCU
	.rcu_read_lock_nesting = 0,
	.rcu_read_unlock_special.s = 0,
	.rcu_node_entry = LIST_HEAD_INIT(init_task.rcu_node_entry),
	.rcu_blocked_node = NULL,
#endif
#ifdef CONFIG_TASKS_RCU
	.rcu_tasks_holdout = false,
	.rcu_tasks_holdout_list = LIST_HEAD_INIT(init_task.rcu_tasks_holdout_list),
	.rcu_tasks_idle_cpu = -1,
#endif
#ifdef CONFIG_TASKS_TRACE_RCU
	.trc_reader_nesting = 0,
	.trc_reader_special.s = 0,
	.trc_holdout_list = LIST_HEAD_INIT(init_task.trc_holdout_list),
	.trc_blkd_node = LIST_HEAD_INIT(init_task.trc_blkd_node),
#endif
#ifdef CONFIG_CPUSETS
	.mems_allowed_seq = SEQCNT_SPINLOCK_ZERO(init_task.mems_allowed_seq,
						 &init_task.alloc_lock),
#endif
#ifdef CONFIG_RT_MUTEXES
	.pi_waiters	= RB_ROOT_CACHED,
	.pi_top_task	= NULL,
#endif
	INIT_PREV_CPUTIME(init_task)
#ifdef CONFIG_VIRT_CPU_ACCOUNTING_GEN
	.vtime.seqcount	= SEQCNT_ZERO(init_task.vtime_seqcount),
	.vtime.starttime = 0,
	.vtime.state	= VTIME_SYS,
#endif
#ifdef CONFIG_NUMA_BALANCING
	.numa_preferred_nid = NUMA_NO_NODE,
	.numa_group	= NULL,
	.numa_faults	= NULL,
#endif
#if defined(CONFIG_KASAN_GENERIC) || defined(CONFIG_KASAN_SW_TAGS)
	.kasan_depth	= 1,
#endif
#ifdef CONFIG_KCSAN
	.kcsan_ctx = {
		.scoped_accesses	= {LIST_POISON1, NULL},
	},
#endif
#ifdef CONFIG_TRACE_IRQFLAGS
	.softirqs_enabled = 1,
#endif
#ifdef CONFIG_LOCKDEP
	.lockdep_depth = 0, /* no locks held yet */
	.curr_chain_key = INITIAL_CHAIN_KEY,
	.lockdep_recursion = 0,
#endif
#ifdef CONFIG_FUNCTION_GRAPH_TRACER
	.ret_stack		= NULL,
	.tracing_graph_pause	= ATOMIC_INIT(0),
#endif
#if defined(CONFIG_TRACING) && defined(CONFIG_PREEMPTION)
	.trace_recursion = 0,
#endif
#ifdef CONFIG_LIVEPATCH
	.patch_state	= KLP_UNDEFINED,
#endif
#ifdef CONFIG_SECURITY
	.security	= NULL,
#endif
#ifdef CONFIG_SECCOMP_FILTER
	.seccomp	= { .filter_count = ATOMIC_INIT(0) },
#endif
};
EXPORT_SYMBOL(init_task);
```

init_task拥有大量的init_xxx变量，这里面的命名空间ns也一般都是系统中其他进程后面会共享的。

初始化完成后，会有两个后继，一个是init进程，一个是kthreadd内核线程。
- init进程是第一个用户进程
- kthreadd负责内核线程

二者都是idle进程在rest_init函数中调用kernel_thread创建的。


## 疑问

1. 内核线程没有自己管理的内存，那运行时的内存是哪里来的？
    
    AI生成，有待确认：内核线程会借用用户线程的mm_struct，但并不使用其中用户部分的，而是借用mm_struct中的页表目录，因为内核空间的虚拟内存映射是通用的。而内核地址部分是所有进程共享的，所以正好。而且这样也能减少页表的切换。
2. vfork使用CLONE_VM共享了进程的mm_struct，甚至包括栈。那么如果某一方退出当前函数，这个栈不是也会销毁，那另一个进程不会出问题吗？或者两者都需要增长栈，这感觉很难实现？

<!-- 进度 电子书209 /纸质197 -->

<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/202/  在线阅读 -->

<!-- https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt -->

<!-- https://elixir.bootlin.com/linux/v6.2.16/source -->

<!-- 本文为第二部分，即电子书201页之后，纸质书189页之后。 -->