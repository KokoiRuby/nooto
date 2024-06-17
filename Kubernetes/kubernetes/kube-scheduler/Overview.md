:confused: **What?**

- Decide workload on which node (update NodeName field).
- [Flags](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
- Supports multiple → `pod.spec.schedulerName`



:confused: **vs.**

- request is ref. to scheduling.

  - multi-initContaiers, take max.

- limit is ref. to cgroup.

  - [LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/)

  



:confused: **[Phase](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)?**

- PreEnqueue: pod marked as Ready for scheduling → into plugins chains → enter active quque only when all plugins return Success, otherwise marked Unscheduled.
- **Scheduling cycle**: selects a node for the Pod (in serial).
- **Binding cycle**: applies that decision (concurrent).
- Both can be aborted if unschedulable/internal err, pod back to queue & retry.



![img](https://kubernetes.io/images/docs/scheduling-framework-extensions.png)



:confused: **[Config](https://kubernetes.io/docs/reference/scheduling/config/)?** Since v1.25

- Each stage is exposed in an extension point ← plugin impl.



:construction_worker: **Production**

- Small cluster around 100 nodes, 8000 pods → 2min.

- When node has issues in low load, pod will be more easily to schedule on low load node, pod creation will fail.

- Fork bomb, too many open files → crash node，pods will be evicted to other nodes & crash them as well. 

  - :smile: [tini](https://github.com/krallin/tini)

  
