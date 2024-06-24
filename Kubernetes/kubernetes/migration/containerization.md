:confused: **Target?** → Stability, Avaialbility, Performance, Security.

- **Instance**: Resource, Configuration, Persistence, Logging & Metrics.

- **Application**: Redundancy, Replica, LB, HC, SD, Monitoring, Failover, Scale.

  - Startup: Boot time, Args.
  - Dockerfile: BaseImage, Utility/Tool, Proc# (parent/children), Decouple conf from code, Entrypoint.

  ↑

- **Security**: Image, Applicati on, Data, Communication.



:confused: **Risk?**

- **CR Log Driver** blocking/non-block mode.
  - Proc stdout/stderr → Buffer ← **CR Log Driver** → dump to log.
  - In blocking mode, disk flush will block application writes to Buffer.
- Shared **Kernel**.
  - Sysctl, PID (fork bomb), FD, Disk



:confused: **Resource Monitoring?**

- Container's scope → Host

  - top, /proc/[cpu|mem]info, df

  - `/sys/fs/cgroup/[cpu|mem]/kubepods.slice`

    - CPU (CPU# = quota/period)
      - `/sys/fs/cgroup/cpu/cpu.cfs_quota_us` (-1 if BestEffort)
      - `/sys/fs/cgroup/cpu/cpu.cfs_period_us`
      - `/sys/fs/cgroup/cpuacct/cpuacct.usage`
      - `/sys/fs/cgroup/cpuacct/cpuacct.usage_percpu` (per CPU)

    - Mem
      - `/sys/fs/cgroup/memory/memory.limit_in_bytes`
      - `/sys/fs/cgroup/memory/memory.usage_in_bytes`

- Misc: [lxcfs](https://github.com/lxc/lxcfs), [Kata](https://katacontainers.io/), [Virtlet](https://github.com/Mirantis/virtlet)