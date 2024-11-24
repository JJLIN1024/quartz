---
title: task_struct
date: 2024-03-22
lastmod: 2024-03-22
author:
  - Jimmy Lin
tags:
  - process
  - thread
  - docker
  - container
  - file_system
draft: false
---
- [task_struct(v6.11.7)](https://elixir.bootlin.com/linux/v6.11.7/source/include/linux/sched.h#L758)
A Process(Thread) in [Linux Kernel](https://elixir.bootlin.com/linux/latest/source/include/linux/sched.h#L748):

```cpp
struct task_struct {
#ifdef CONFIG_THREAD_INFO_IN_TASK
	/*
	 * For reasons of header soup (see current_thread_info()), this
	 * must be the first element of task_struct.
	 */
	struct thread_info		thread_info;
#endif
	struct mm_struct		*mm;
	...
	pid_t				pid;
	...
	/*
	 * Pointers to the (original) parent process, youngest child, younger sibling,
	 * older sibling, respectively.  (p->father can be replaced with
	 * p->real_parent->pid)
	 */

	/* Real parent process: */
	struct task_struct __rcu	*real_parent;

	/* Recipient of SIGCHLD, wait4() reports: */
	struct task_struct __rcu	*parent;

	/*
	 * Children/sibling form the list of natural children:
	 */
	struct list_head		children;
	struct list_head		sibling;
	struct task_struct		*group_leader;
	...
	/* Filesystem information: */
	struct fs_struct		*fs;

	/* Open file information: */
	struct files_struct		*files;
	...
	...
	/* Namespaces: */
	struct nsproxy			*nsproxy;
	...
	...
	/* Control Group info protected by css_set_lock: */
	struct css_set __rcu		*cgroups;
	/* cg_list protected by css_set_lock and tsk->alloc_lock: */
	struct list_head		cg_list;
	...
	/* CPU-specific state of this task: */
	struct thread_struct		thread;
};
```

## Memory (mm_struct)

- [Memory mapping](https://linux-kernel-labs.github.io/refs/pull/222/merge/labs/memory_mapping.html)
![[Pasted image 20240322112006.png]]
> [image source](https://medium.com/@boutnaru/linux-kernel-data-structures-struct-vm-area-struct-2e832914865d)

![[Pasted image 20240322124118.png]]
> [image source](https://images0.cnblogs.com/blog/516769/201304/13214813-ece9b1c2abbd4ea8b1de1dd266849b73.png)

Before v6.1-rc1:

VMA（Virtual Memory Area）也用紅黑樹來紀錄追蹤頁面 (page) 變更，因為後者不免存在頻繁的讀取 VMA 結構，如 page fault 和 mmap 等操作，且當大量的已對應 (mapped) 區域時存在時，若要尋找某個特定的虛擬記憶體地址，鏈結串列 (linked list) 的走訪成本過高，因此需要一種資料結構以提供更有效率的尋找，於是紅黑樹就可勝任。
>  [Linux 核心的紅黑樹](https://hackmd.io/@sysprog/linux-rbtree)

- [[Red Black Tree]]

`vm_area_struct` is a doubly linked list, and `rb_root` is red black tree for faster query of a particular virtual memory address.

`pgd` is the address of the global page table.

![[Pasted image 20240322134511.png]]
> [image source](https://www.ffutop.com/posts/2019-07-17-understand-kernel-13/)

- `start_code`、 `end_code`：starting and ending memory address of the code section.
- `start_data`、 `end_data`：starting and ending memory address of the data section.
- `start_stack`：starting memory address of the stack section.
- `start_brk` 、 `brk` ：starting and ending memory address of the heap section.
- `arg_start`、`arg_end`：starting and ending memory address of the function arguments.
- `env_start`, `env_end` ：starting and ending memory address of the environmental arguments.

```c
struct mm_struct {
	struct {
		struct vm_area_struct *mmap;
		struct rb_root mm_rb;
		...
		pgd_t * pgd;
		...
		unsigned long start_code, end_code, start_data, end_data;
		unsigned long start_brk, brk, start_stack;
		unsigned long arg_start, arg_end, env_start, env_end;
};
```

```c
struct vm_area_struct {
	/* The first cache line has the info for VMA tree walking. */

	unsigned long vm_start;		/* Our start address within vm_mm. */
	unsigned long vm_end;		/* The first byte after our end address
					   within vm_mm. */

	/* linked list of VM areas per task, sorted by address */
	struct vm_area_struct *vm_next, *vm_prev;
	...
	/*
	 * Access permissions of this VMA.
	 * See vmf_insert_mixed_prot() for discussion.
	 */
	pgprot_t vm_page_prot;
	unsigned long vm_flags;		/* Flags, see mm.h. */

```

Since v6.1-rc1:
- [mm: remove rb tree.](https://github.com/torvalds/linux/commit/524e00b36e8c547f5582eef3fb645a8d9fc5e3df)：Remove the RB tree and start using the [maple tree](https://docs.kernel.org/core-api/maple_tree.html) for `vm_area_struct` tracking.

```c
struct mm_struct {
	struct {
		struct maple_tree mm_mt;
		...
		pgd_t * pgd;
		...
		unsigned long start_code, end_code, start_data, end_data;
		unsigned long start_brk, brk, start_stack;
		unsigned long arg_start, arg_end, env_start, env_end;
};
```

```c
/*
 * If the tree contains a single entry at index 0, it is usually stored in
 * tree->ma_root.  To optimise for the page cache, an entry which ends in '00',
 * '01' or '11' is stored in the root, but an entry which ends in '10' will be
 * stored in a node.  Bits 3-6 are used to store enum maple_type.
 *
 * The flags are used both to store some immutable information about this tree
 * (set at tree creation time) and dynamic information set under the spinlock.
 *
 * Another use of flags are to indicate global states of the tree.  This is the
 * case with the MAPLE_USE_RCU flag, which indicates the tree is currently in
 * RCU mode.  This mode was added to allow the tree to reuse nodes instead of
 * re-allocating and RCU freeing nodes when there is a single user.
 */
struct maple_tree {
	union {
		spinlock_t	ma_lock;
		lockdep_map_p	ma_external_lock;
	};
	unsigned int	ma_flags;
	void __rcu      *ma_root;
};

```

```c
/*
 * This struct describes a virtual memory area. There is one of these
 * per VM-area/task. A VM area is any part of the process virtual memory
 * space that has a special rule for the page-fault handlers (ie a shared
 * library, the executable area etc).
 */
struct vm_area_struct {
	/* The first cache line has the info for VMA tree walking. */

	union {
		struct {
			/* VMA covers [vm_start; vm_end) addresses within mm */
			unsigned long vm_start;
			unsigned long vm_end;
		};
#ifdef CONFIG_PER_VMA_LOCK
		struct rcu_head vm_rcu;	/* Used for deferred freeing. */
#endif
	};
```

- [Linux 核心設計: RCU 同步機制](https://hackmd.io/@sysprog/linux-rcu)
- [The Maple Tree, A Modern Data Structure for a Complex Problem](https://blogs.oracle.com/linux/post/the-maple-tree-a-modern-data-structure-for-a-complex-problem)
- [The Maple Tree - Liam Howlett](https://youtu.be/eKuTTpmllZ8?si=3Icl_zHjqwrNxNVv)
### Per Thread's Stack

`sp` holds the start address of the stack pointer of the thread's stack. 

```cpp
struct thread_struct {
	/* Cached TLS descriptors: */
	struct desc_struct	tls_array[GDT_ENTRY_TLS_ENTRIES];
#ifdef CONFIG_X86_32
	unsigned long		sp0;
#endif
	unsigned long		sp;
	...
```
> [x86 中的 thread_struct](https://elixir.bootlin.com/linux/latest/source/arch/x86/include/asm/processor.h#L429)

### Process V.S. Thread

![](https://miro.medium.com/v2/resize:fit:700/1*u3RS66CduOnZFYHY75lBLQ.png)

如果產生出來的 `task_struct` 有自己的 `mm_struct`，稱為 Process，其實本身就是個 Main Thread (CPU scheduling的基本單位)。如果 產生出來的 `task_struct` 共用別人的 `mm_struct`，稱為 Thread。

The Linux kernel does not provide any special scheduling semantics of data structures to represent threads. Instead, **a thread is merely a process that shares certain resources with other processes**.

若一個 `task_struct` 的 `mm_struct = NULL`，代表這個 `task_struct` 是一個 kernel thread。

## Files

- [Linux 核心設計: 檔案系統概念及實作手法](https://hackmd.io/@sysprog/linux-file-system)
- [File system drivers (Part 1)](https://linux-kernel-labs.github.io/refs/pull/187/merge/labs/filesystems_part1.html)

![[Pasted image 20240326200659.png]]

![[Pasted image 20240322144722.png]]
> [image source](https://images.slideplayer.com/16/5172630/slides/slide_10.jpg)

一個 process(`task_struct)` 有以下兩個 field：

```c
/* Filesystem information: */
struct fs_struct		*fs;

/* Open file information: */
struct files_struct		*files;
```

其中，`fs_struct` 代表這個 process 目前所使用的 file system 的 information，其中，每一個 `path` 都具有路徑（`dentry`）以及 `mount` 的相關資訊，包括 `mount` 在哪裡（`dentry`）以及這個 mounted 的 file system 的資訊（`super_block`）。

```c
struct fs_struct {
	...
	struct path root, pwd;
} __randomize_layout;
```
> root path 和目前的 working directory
> [struct fs_struct](https://elixir.bootlin.com/linux/latest/source/include/linux/fs_struct.h)

```c
struct path {
	struct vfsmount *mnt;
	struct dentry *dentry;
} __randomize_layout;
```

```c
struct vfsmount {
	struct dentry *mnt_root;	/* root of the mounted tree */
	struct super_block *mnt_sb;	/* pointer to superblock */
	...
} __randomize_layout;
```

`dentry` 代表路徑（directory），具有對應的 `inode` 的資訊。
> [source](https://litux.nl/mirror/kerneldevelopment/0672327201/ch12lev1sec7.html)

```c
struct dentry {
	...
	struct dentry *d_parent;	/* parent directory */
	...
	struct inode *d_inode;		/* Where the name belongs to - NULL is
					 * negative */
	...
	const struct dentry_operations *d_op;
	struct super_block *d_sb;	/* The root of the dentry tree */
	...
};

```

`super_block` is the control block of the one specific file system. 可觀察到其具有一些屬性，包括 `dev_t`，也就是 device 的編號。

```c
struct super_block {
	...
	dev_t			s_dev;		/* search index; _not_ kdev_t */
	...
	loff_t			s_maxbytes;	/* Max file size */
	struct file_system_type	*s_type;
	const struct super_operations	*s_op;
	...
};
```

```c
struct super_operations {
   	struct inode *(*alloc_inode)(struct super_block *sb);
	void (*destroy_inode)(struct inode *);
	void (*free_inode)(struct inode *);

   	void (*dirty_inode) (struct inode *, int flags);
	int (*write_inode) (struct inode *, struct writeback_control *wbc);
	int (*drop_inode) (struct inode *);
	...
};
```

`inode` ：inode (index node）是指在許多「類Unix檔案系統」中的一種資料結構，用於描述檔案系統對象（包括檔案、目錄、裝置檔案、socket、管道等）。每個inode儲存了檔案系統對象資料的屬性和磁碟塊位置。檔案系統對象屬性包含了各種元資料（如：最後修改時間） ，也包含使用者群組（owner）和權限資料。
> [source](https://zh.wikipedia.org/zh-tw/Inode)

```c
struct inode {
	...
	const struct inode_operations	*i_op;
	struct super_block	*i_sb;
	...
};
```

```c
struct inode_operations {
	struct dentry * (*lookup) (struct inode *,struct dentry *, unsigned int);
	const char * (*get_link) (struct dentry *, struct inode *, struct delayed_call *);
	int (*permission) (struct mnt_idmap *, struct inode *, int);
	struct posix_acl * (*get_inode_acl)(struct inode *, int, bool);

	int (*readlink) (struct dentry *, char __user *,int);

	int (*create) (struct mnt_idmap *, struct inode *,struct dentry *,
		       umode_t, bool);
	int (*link) (struct dentry *,struct inode *,struct dentry *);
	int (*unlink) (struct inode *,struct dentry *);
	int (*symlink) (struct mnt_idmap *, struct inode *,struct dentry *,
	...
};
		
```

至於一個 process 的 open files，使用 `files_struct` 表示。

![[Pasted image 20240322153029.png]]

![[Pasted image 20240322155124.png]]
```c
/*
 * Open file table structure
 */
struct files_struct {
	...
	struct fdtable __rcu *fdt;
	struct fdtable fdtab;
	...
	struct file __rcu * fd_array[NR_OPEN_DEFAULT];
};
```
> [struct files_struct](https://elixir.bootlin.com/linux/latest/source/include/linux/fdtable.h#L49)

```c
struct fdtable {
	unsigned int max_fds;
	struct file __rcu **fd;      /* current fd array */
	...
};
```

 `fdt` 為 per process 的 open file table（ `fdtable`, file descriptor table），`fdtab` 為其備份。

如果一個 process 打開的 file 數目多於 `NR_OPEN_DEFAULT` （32 bit machine 上是 32個，64 bit 是 64 個），kernel 就會分配一個新的更大的 `fd_array`，並將其地址存放在`fd` 欄位中，而這個 `fd_array` 所包含的 file 數目存放在 `max_fds` 欄位。

![[Pasted image 20240322153438.png]]

一個 file 會具有其對應的 inode pointer。

```c
struct file {
	...
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;
};
```

```c
struct file_operations {
	struct module *owner;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	...
	...
	int (*mmap) (struct file *, struct vm_area_struct *);
	...
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);
	...
};
```

## Namespaces and "containers"
- [[Docker]]

A `task_struct` contains the following two fields, which is essential for implementing linux containers:

```c
/* Namespaces: */
struct nsproxy			*nsproxy;
...
...
/* Control Group info protected by css_set_lock: */
struct css_set __rcu		*cgroups;
```

```c
/*
 * A structure to contain pointers to all per-process
 * namespaces - fs (mount), uts, network, sysvipc, etc.
 *
 * The pid namespace is an exception -- it's accessed using
 * task_active_pid_ns.  The pid namespace here is the
 * namespace that children will use.
 *
 * 'count' is the number of tasks holding a reference.
 * The count for each namespace, then, will be the number
 * of nsproxies pointing to it, not the number of tasks.
 *
 * The nsproxy is shared by tasks which share all namespaces.
 * As soon as a single namespace is cloned or unshared, the
 * nsproxy is copied.
 */
struct nsproxy {
	refcount_t count;
	struct uts_namespace *uts_ns;
	struct ipc_namespace *ipc_ns;
	struct mnt_namespace *mnt_ns;
	struct pid_namespace *pid_ns_for_children;
	struct net 	     *net_ns;
	struct time_namespace *time_ns;
	struct time_namespace *time_ns_for_children;
	struct cgroup_namespace *cgroup_ns;
};
```

see [User Namespace 詳解](https://tinylab.org/user-namespace/)

[Namespaces in operation, part 1: namespaces overview](https://lwn.net/Articles/531114/)

[[css_set]]
[[nsproxy]]
