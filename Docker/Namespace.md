:confused: **What is Linux Namespace?**

- A kernel feature that wrap proc resource which isolates to another.



:confused: **[source code](https://github.com/torvalds/linux)？**

```c
/* ./include/linux/sched.h */
struct task_struct {
    ...
    /* Namespaces: */
    struct nsproxy *nsproxy;
    ...
}

/* ./include/linux/nsproxy.h */
struct nsproxy {
	refcount_t count;
	struct uts_namespace *uts_ns;
	struct ipc_namespace *ipc_ns;
	struct mnt_namespace *mnt_ns;
	struct pid_namespace *pid_ns_for_children;
	struct net *net_ns;
	struct time_namespace *time_ns;
	struct time_namespace *time_ns_for_children;
	struct cgroup_namespace *cgroup_ns;
};
```



:confused: **sys call?**

- **clone**: creates a child proc and allows it share certain resources with parent.

- **setns**: allows a proc to join an existing namespace by a fd

- **unshare**: allows a proc to disassociate itself from certain namespaces

  

:confused: **Namespaces?**

| Type   | Isolation                            | Kernel ver. |
| ------ | ------------------------------------ | ----------- |
| mnt    | mnt fs view                          | 2.4.19      |
| uts    | hostname, domain name                | 2.6.19      |
| ipc    | ipc                                  | 2.6.19      |
| pid    | pid                                  | 2.4.24      |
| net    | nic, ip addr, route table, /proc/net | 2.4.24      |
| usr    | uid, gid                             | 3.8         |
| cgroup | cgroup                               | 4.6         |



:bookmark_tabs: **Cheatsheet**

```bash
# unshare ns from parent
$ unshare <type_flag> <cmd>

# list ns
$ lsns
$ lsns -t <type>

# ls proc ns
$ ls -la /proc/<pid>/ns/

# into ns
$ nsenter -t <pid> <type flag> <cmd>
```

