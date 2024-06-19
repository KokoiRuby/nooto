:confused: **How to secure workload HA?**

- Reasonable limit, avoid OOM killed.
- Reasonable emptyDir.sizeLimit & make sure no exceed, otherwise evicted.



:confused: **QoS?**

- Eviction order: BestEffort > Burstable > Guaranteed.



:confused: **Is Guaranteed always better?**

- No, node utilization ↓
- Burstable → Overselling.



<img src="https://techiescamp.com/wp-content/uploads/2023/02/image-14-691x1024.png" alt="img" style="zoom: 80%;" />



:confused: **Taint-based Eviction?**

- effect: NoSchedule/NoExecute
- When?
  - network partition
  - kubelet, containerd not working
  - node reboot over 15min
- :construction_worker: increase tolerationSeconds to avoid eviction, esp. stateful app with local storage.