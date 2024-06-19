:confused: **Resources?**

- CPU/GPU/Mem each workload?
- Need overselling?
- Storage for each workload?
  - size
  - local | network
  - R/W perf
  - Disk IO
- Network: overall QPS & Bandwidth



:confused: **Challenge from storage?**

- multi-container can shared emptyDir
  - set sizeLimit otherwise risk in root disk
  - regular du from kubelet → perf impact
  - evict if exceeded, everthing will lose



:confused: **Where to keep data?**

| Type           | Persistent after container restart | Persistent after pod restart | size control? | Attention             |
| -------------- | ---------------------------------- | ---------------------------- | ------------- | --------------------- |
| emptyDir       | Y                                  | N                            | Y             |                       |
| hostPath       | Y                                  | N                            | N             | Permission            |
| Local volume   | Y                                  | N                            | Y             | No backup             |
| Network volume | Y                                  | Y                            | Y             |                       |
| rootFS         | N                                  | N                            | N             | DON'T Write Anything! |



<img src="./Practice.assets/image-20240619085454998.png" alt="image-20240619085454998" style="zoom: 67%;" />



:confused: **Conf?**

- src: cm/secret/download API → env/vol mnt



:confused: **HA?**

- replicas?
- Rolling Update strategy?
- PodTemplateHash



:confused: **When container would be possibly killed?**

| Scenario              | Effect                         | Suggestion                                                   |
| --------------------- | ------------------------------ | ------------------------------------------------------------ |
| kubelet upgdate       | pod restart                    | redundanct/cross-regison deployment                          |
| os update/node reboot | pod terminated for minutes     | redundanct/cross-regison deployment<br />probe<br />tolerationSeconds |
| node off-rack         | node drain<br />pod terminated | redundanct/cross-regison deployment<br />pod disruption budget<br />preStop to perform backup |
| node crash            | pod terminated                 | redundanct/cross-regison deployment<                         |