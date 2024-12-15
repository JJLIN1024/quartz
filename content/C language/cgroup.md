---
title: cgroup
date: 2024-11-10
lastmod: 2024-11-10
author:
  - Jimmy Lin
tags: 
draft: false
---
- [kernel doc: cgroup_v1](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt)
- [kernel doc: cgroup_v2](https://www.kernel.org/doc/Documentation/cgroup-v2.txt
- [Control Groups 解說](https://hackmd.io/@vax-r/cgroups?utm_source=preview-mode&utm_medium=rec)

- [[cgroup_subsys_state]]

```c
struct cgroup {
	/* self css with NULL ->ss, points back to this cgroup */
	struct cgroup_subsys_state self;

	unsigned long flags;		/* "unsigned long" so bitops work */

	/*
	 * The depth this cgroup is at.  The root is at depth zero and each
	 * step down the hierarchy increments the level.  This along with
	 * ancestors[] can determine whether a given cgroup is a
	 * descendant of another without traversing the hierarchy.
	 */
	int level;

	/* Maximum allowed descent tree depth */
	int max_depth;
// ...
	/* Private pointers for each registered subsystem */
	struct cgroup_subsys_state __rcu *subsys[CGROUP_SUBSYS_COUNT];

	struct cgroup_root *root;
```