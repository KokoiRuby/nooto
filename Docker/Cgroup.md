:confused: **What is Cgroup?**

- Control Group, a kernel feature to allocate, isolate resource & limit usage of proc.
  - subsystem `/sys/fs/cgroup`


| Resource | Desc                      |
| -------- | ------------------------- |
| blkio    | blk dev io                |
| cpu      | scheduler for cpu quantum |
| cpuacct  | cpu accounting            |
| cpuset   | multi-core, dedicate cpu  |
| devices  | dev access/deny           |
| freezer  | stop/resume               |
| memory   | mem & report              |
| net_cls  | packet classify           |
| ns       | namespace                 |
| pid      | pid                       |



:confused: **cpu subsys?**

- shares
  - CgroupA share: 512 ; (33%) 
  - CgroupB share: 1024; (66%)
- quota vs. period
  - sum(quota) <= period

|                   | Desc                                                    |
| ----------------- | ------------------------------------------------------- |
| cpu.shares        | relative weight of the cgroup in CPU sharing mode       |
| cpu.cfs_period_us | period length of scheduling period                      |
| cpu.cfs_quota_us  | amount of CPU time can use within the scheduling period |
| cpu.stat          | cpu statistics                                          |
| nr_periods        | number of cpu.cfs_period_us                             |
| nr_throttled      | number of times the cgroup has been throttled           |
| throttled_time    | total time duration of throttled                        |



:confused: **cpuacct subsys?**

|               | Desc                                           |
| ------------- | ---------------------------------------------- |
| cpuacct.usage | total CPU time used by all tasks in the cgroup |
| cpuacct.stat  | stats of all tasks in the cgroup               |



:confused: **Linux CFS?**

- Completely-Fair Scheduler.
-  CPU time is distributed fairly among all runnable tasks based on relative weight.
  - Higher weights receive more CPU time
- ++ **vRuntime** = Runtime * 1024 / task_weight; smaller vRuntime given precedence.
- Tasks **in Red/Black Tree**
  - pick leftmost = smallest vRuntime
  - __scheduler() → context_switch()



:bookmark_tabs: **Hands-on**

```bash
$ mkdir /sys/fs/cgroup/cpu/foo && ll /sys/fs/cgroup/cpu/foo

# python loop to cost
$ nohup bash -c 'while true; do echo "Hello, world!"; done' >/dev/null 2>&1 &

# check process cpu -> %CPU is 100% -> cost 1 CPU core
$ top

# + proc into cgroup
$ echo <pid> > /sys/fs/cgroup/cpu/foo/tasks

# limit to 10% CPU
# default cpu.cfs_quota_us is 100000 us
$ echo 10000 > /sys/fs/cgroup/cpu/foo/cpu.cfs_quota_us

# check again
$ top

# clean
$ cgdelete cpu:foo
```



:confused: **mem subsys?**

|                    | Desc                                                |
| ------------------ | --------------------------------------------------- |
| usage_in_bytes     | amount of memory currently consumed in cgroup       |
| max_usage_in_bytes | max amount of memory currently consumed in cgroup   |
| limit_in_bytes     | limit amount of memory currently consumed in cgroup |
| oom_control        |                                                     |



:bookmark_tabs: **Hands-on**

```bash
# install
$ sudo apt-get install -y cgroup-tools

# set limit
$ sudo cgcreate -g memory:bar
$ sudo cgset -r memory.limit_in_bytes=100M bar

# chk
$ cat /sys/fs/cgroup/memory/bar/memory.limit_in_bytes

# script
$ cat > consume_memory.sh << EOF
#!/bin/bash
while true; do
    arr+=("$(yes | head -n 100000)")
done
EOF

# run
$ chmod +x consume_memory.sh && sudo cgexec -g memory:bar ./consume_memory.sh

# watch oom
$ sudo dmesg | grep -i oom

# clean
$ cgdelete memory:bar
```



:confused: **Cgroup Driver conflict?**

- systemd as init → cgroup per unit.
- cgroupfs → default in docker (if as CR for kubelet).
