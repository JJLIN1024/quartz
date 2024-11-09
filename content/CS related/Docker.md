---
title: Docker
date: 2024-03-19
lastmod: 2024-03-19
author:
  - Jimmy Lin
tags:
  - linux_kernel
  - container
draft: false
---
- [《Docker —— 從入門到實踐­》正體中文版](https://philipzheng.gitbook.io/docker_practice)

## Docker 基本操作

更簡單的，建議大家下載 [.bashrc_docker](https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker)，並將內容放到 .bashrc 中。

```
$ wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
$ echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
```

這個檔案中定義了很多方便使用 Docker 的命令，例如 `docker-pid` 可以取得某個容器的 PID；而 `docker-enter` 可以進入容器或直接在容器內執行命令。

## Docker Internal

- [Building a container from scratch in Go - Liz Rice (Microscaling Systems)](https://youtu.be/Utf-A4rODH8?si=MhhXrEEQ2oIB4i_Y)
- [unshare(1) — Linux manual page](https://man7.org/linux/man-pages/man1/unshare.1.html)
- Namespace
	- [系統程式設計 - Namespaces](https://hackmd.io/@0xff07/r1wCFz0ut)
	- [namespaces(7) — Linux manual page](https://man7.org/linux/man-pages/man7/namespaces.7.html)
	- [How Docker Works - Intro to Namespaces](https://youtu.be/-YnMr1lj4Z8?si=h5cqBT9tN7aoRc4y)

Namespaces are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources.

```console
jimmylin@ubuntu  ~  id
uid=1000(jimmylin) gid=1000(jimmylin) groups=1000(jimmylin),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
 jimmylin@ubuntu  ~  unshare --user --pid --map-root-user --mount-proc --fork bash
 ⚡ root@ubuntu  ~  ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 05:23 pts/0    00:00:00 bash
root         103       1  0 05:23 pts/0    00:00:00 ps -ef
 ⚡ root@ubuntu  ~  id
uid=0(root) gid=0(root) groups=0(root),65534(nogroup)
```

If we don't mount the /proc file system, we can see all processes.

```console
 jimmylin@ubuntu  ~  unshare --user --pid --map-root-user --fork bash
 ⚡ root@ubuntu  ~  ps
    PID TTY          TIME CMD
   1016 pts/0    00:00:00 bash
   1789 pts/0    00:00:00 unshare
   1790 pts/0    00:00:00 bash
   1891 pts/0    00:00:00 ps
 ⚡ root@ubuntu  ~  ps -ef | head -5
UID          PID    PPID  C STIME TTY          TIME CMD
nobody         1       0  0 05:17 ?        00:00:00 /sbin/init
nobody         2       0  0 05:17 ?        00:00:00 [kthreadd]
nobody         3       2  0 05:17 ?        00:00:00 [rcu_gp]
nobody         4       2  0 05:17 ?        00:00:00 [rcu_par_gp]
```


The kernel assigns each process a symbolic link per namespace kind in `/proc/<pid>/ns/`. The inode number pointed to by this symlink is the same for each process in this namespace. This uniquely identifies each namespace by the inode number pointed to by one of its symlinks.

```console
 jimmylin@ubuntu  ~  ls -la /proc/$$/ns/
total 0
dr-x--x--x 2 jimmylin jimmylin 0 Mar 19 05:27 .
dr-xr-xr-x 9 jimmylin jimmylin 0 Mar 19 05:27 ..
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 net -> 'net:[4026531840]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:37 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 time -> 'time:[4026531834]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:37 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 user -> 'user:[4026531837]'
lrwxrwxrwx 1 jimmylin jimmylin 0 Mar 19 05:27 uts -> 'uts:[4026531838]'
```
> We can see there are symlinks

Reading the symlink via `readlink` returns a string containing the namespace kind name and the inode number of the namespace.

A control group (cgroup) is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network, and so on) of a collection of processes.

- [What Are Namespaces and cgroups, and How Do They Work?](https://www.nginx.com/blog/what-are-namespaces-cgroups-how-do-they-work/)
- [Control Groups¶](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html)
- [Control Group v2](https://docs.kernel.org/admin-guide/cgroup-v2.html?highlight=namespace)

- `css_set` within the `task_struct`
- [cgroup原始碼分析1—— css_set和cgroup的關係](https://linux.laoqinren.net/kernel/cgroup-source-css_set-and-cgroup/)