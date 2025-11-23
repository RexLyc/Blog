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
	retval = copy_thread(p, args);
	if (retval)
		goto bad_fork_cleanup_io;

	stackleak_task_init(p);

	if (pid != &init_struct_pid) {
		pid = alloc_pid(p->nsproxy->pid_ns_for_children, args->set_tid,
				args->set_tid_size);
		if (IS_ERR(pid)) {
			retval = PTR_ERR(pid);
			goto bad_fork_cleanup_thread;
		}
	}

	/*
	 * This has to happen after we've potentially unshared the file
	 * descriptor table (so that the pidfd doesn't leak into the child
	 * if the fd table isn't shared).
	 */
	if (clone_flags & CLONE_PIDFD) {
		retval = get_unused_fd_flags(O_RDWR | O_CLOEXEC);
		if (retval < 0)
			goto bad_fork_free_pid;

		pidfd = retval;

		pidfile = anon_inode_getfile("[pidfd]", &pidfd_fops, pid,
					      O_RDWR | O_CLOEXEC);
		if (IS_ERR(pidfile)) {
			put_unused_fd(pidfd);
			retval = PTR_ERR(pidfile);
			goto bad_fork_free_pid;
		}
		get_pid(pid);	/* held by pidfile now */

		retval = put_user(pidfd, args->pidfd);
		if (retval)
			goto bad_fork_put_pidfd;
	}

#ifdef CONFIG_BLOCK
	p->plug = NULL;
#endif
	futex_init_task(p);

	/*
	 * sigaltstack should be cleared when sharing the same VM
	 */
	if ((clone_flags & (CLONE_VM|CLONE_VFORK)) == CLONE_VM)
		sas_ss_reset(p);

	/*
	 * Syscall tracing and stepping should be turned off in the
	 * child regardless of CLONE_PTRACE.
	 */
	user_disable_single_step(p);
	clear_task_syscall_work(p, SYSCALL_TRACE);
#if defined(CONFIG_GENERIC_ENTRY) || defined(TIF_SYSCALL_EMU)
	clear_task_syscall_work(p, SYSCALL_EMU);
#endif
	clear_tsk_latency_tracing(p);

	/* ok, now we should be set up.. */
	p->pid = pid_nr(pid);
	if (clone_flags & CLONE_THREAD) {
		p->group_leader = current->group_leader;
		p->tgid = current->tgid;
	} else {
		p->group_leader = p;
		p->tgid = p->pid;
	}

	p->nr_dirtied = 0;
	p->nr_dirtied_pause = 128 >> (PAGE_SHIFT - 10);
	p->dirty_paused_when = 0;

	p->pdeath_signal = 0;
	INIT_LIST_HEAD(&p->thread_group);
	p->task_works = NULL;
	clear_posix_cputimers_work(p);

#ifdef CONFIG_KRETPROBES
	p->kretprobe_instances.first = NULL;
#endif
#ifdef CONFIG_RETHOOK
	p->rethooks.first = NULL;
#endif

	/*
	 * Ensure that the cgroup subsystem policies allow the new process to be
	 * forked. It should be noted that the new process's css_set can be changed
	 * between here and cgroup_post_fork() if an organisation operation is in
	 * progress.
	 */
	retval = cgroup_can_fork(p, args);
	if (retval)
		goto bad_fork_put_pidfd;

	/*
	 * Now that the cgroups are pinned, re-clone the parent cgroup and put
	 * the new task on the correct runqueue. All this *before* the task
	 * becomes visible.
	 *
	 * This isn't part of ->can_fork() because while the re-cloning is
	 * cgroup specific, it unconditionally needs to place the task on a
	 * runqueue.
	 */
	sched_cgroup_fork(p, args);

	/*
	 * From this point on we must avoid any synchronous user-space
	 * communication until we take the tasklist-lock. In particular, we do
	 * not want user-space to be able to predict the process start-time by
	 * stalling fork(2) after we recorded the start_time but before it is
	 * visible to the system.
	 */
    // 这里再次更新时间
	p->start_time = ktime_get_ns();
	p->start_boottime = ktime_get_boottime_ns();

	/*
	 * Make it visible to the rest of the system, but dont wake it up yet.
	 * Need tasklist lock for parent etc handling!
	 */
	write_lock_irq(&tasklist_lock);

	/* CLONE_PARENT re-uses the old parent */
	if (clone_flags & (CLONE_PARENT|CLONE_THREAD)) {
		p->real_parent = current->real_parent;
		p->parent_exec_id = current->parent_exec_id;
		if (clone_flags & CLONE_THREAD)
			p->exit_signal = -1;
		else
			p->exit_signal = current->group_leader->exit_signal;
	} else {
		p->real_parent = current;
		p->parent_exec_id = current->self_exec_id;
		p->exit_signal = args->exit_signal;
	}

	klp_copy_process(p);

	sched_core_fork(p);

	spin_lock(&current->sighand->siglock);

	rv_task_fork(p);

	rseq_fork(p, clone_flags);

	/* Don't start children in a dying pid namespace */
	if (unlikely(!(ns_of_pid(pid)->pid_allocated & PIDNS_ADDING))) {
		retval = -ENOMEM;
		goto bad_fork_cancel_cgroup;
	}

	/* Let kill terminate clone/fork in the middle */
	if (fatal_signal_pending(current)) {
		retval = -EINTR;
		goto bad_fork_cancel_cgroup;
	}

	/* No more failure paths after this point. */

	/*
	 * Copy seccomp details explicitly here, in case they were changed
	 * before holding sighand lock.
	 */
	copy_seccomp(p);

	init_task_pid_links(p);
	if (likely(p->pid)) {
		ptrace_init_task(p, (clone_flags & CLONE_PTRACE) || trace);

		init_task_pid(p, PIDTYPE_PID, pid);
		if (thread_group_leader(p)) {
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
		attach_pid(p, PIDTYPE_PID);
		nr_threads++;
	}
	total_forks++;
	hlist_del_init(&delayed.node);
	spin_unlock(&current->sighand->siglock);
	syscall_tracepoint_update(p);
	write_unlock_irq(&tasklist_lock);

	if (pidfile)
		fd_install(pidfd, pidfile);

	proc_fork_connector(p);
	sched_post_fork(p);
	cgroup_post_fork(p, args);
	perf_event_fork(p);

	trace_task_newtask(p, clone_flags);
	uprobe_copy_process(p, clone_flags);

	copy_oom_score_adj(clone_flags, p);

	return p;

bad_fork_cancel_cgroup:
	sched_core_free(p);
	spin_unlock(&current->sighand->siglock);
	write_unlock_irq(&tasklist_lock);
	cgroup_cancel_fork(p, args);
bad_fork_put_pidfd:
	if (clone_flags & CLONE_PIDFD) {
		fput(pidfile);
		put_unused_fd(pidfd);
	}
bad_fork_free_pid:
	if (pid != &init_struct_pid)
		free_pid(pid);
bad_fork_cleanup_thread:
	exit_thread(p);
bad_fork_cleanup_io:
	if (p->io_context)
		exit_io_context(p);
bad_fork_cleanup_namespaces:
	exit_task_namespaces(p);
bad_fork_cleanup_mm:
	if (p->mm) {
		mm_clear_owner(p->mm, p);
		mmput(p->mm);
	}
bad_fork_cleanup_signal:
	if (!(clone_flags & CLONE_THREAD))
		free_signal_struct(p->signal);
bad_fork_cleanup_sighand:
	__cleanup_sighand(p->sighand);
bad_fork_cleanup_fs:
	exit_fs(p); /* blocking */
bad_fork_cleanup_files:
	exit_files(p); /* blocking */
bad_fork_cleanup_semundo:
	exit_sem(p);
bad_fork_cleanup_security:
	security_task_free(p);
bad_fork_cleanup_audit:
	audit_free(p);
bad_fork_cleanup_perf:
	perf_event_free_task(p);
bad_fork_cleanup_policy:
	lockdep_free_task(p);
#ifdef CONFIG_NUMA
	mpol_put(p->mempolicy);
#endif
bad_fork_cleanup_delayacct:
	delayacct_tsk_free(p);
bad_fork_cleanup_count:
	dec_rlimit_ucounts(task_ucounts(p), UCOUNT_RLIMIT_NPROC, 1);
	exit_creds(p);
bad_fork_free:
	WRITE_ONCE(p->__state, TASK_DEAD);
	exit_task_stack_account(p);
	put_task_stack(p);
	delayed_free_task(p);
fork_out:
	spin_lock_irq(&current->sighand->siglock);
	hlist_del_init(&delayed.node);
	spin_unlock_irq(&current->sighand->siglock);
	return ERR_PTR(retval);
}
```

<!-- 进度 电子书209 /纸质197 -->

<!-- 可从https://fliphtml5.com/ytimv/nlep/%E5%9B%BE%E8%A7%A3Linux%E5%86%85%E6%A0%B8%EF%BC%88%E5%9F%BA%E4%BA%8E6.x%EF%BC%89_%28%E5%A7%9C%E4%BA%9A%E5%8D%8E%29_%28Z-Library%29/202/  在线阅读 -->

<!-- https://elixir.bootlin.com/linux/v5.0/source/Documentation/x86/x86_64/mm.txt -->

<!-- https://elixir.bootlin.com/linux/v6.2.16/source -->

<!-- 本文为第二部分，即电子书201页之后，纸质书189页之后。 -->