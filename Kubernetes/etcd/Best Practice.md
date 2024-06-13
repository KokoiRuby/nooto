:pencil2: **HA** is top priority. All W hit to Leader.

- How many peers?
- Current R ↑ if more peer?
- Need flex up (scale)?



:pencil2: **Storage**

- Remote vs. Local SSD (better).
- [Size](https://etcd.io/docs/v3.5/dev-guide/limit/)? 



:pencil2: **Security**

- peer2peer (if truly needed ? as cost & hard to maintain).
- k8s secret for [data encryption](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).



:pencil2: **Event Separation**

- pressure ↓
- kube-apiserver Flag: `--etcd-servers-overrides`



:pencil2: **Network Latency**

- Same geo.
- MsgProp dropped from Follower to Leader if Leader in overload.



:pencil2: **Disk IO**

- Dedicated SSD
- inoice wigh high prio if shared with others.



:pencil2: **Log**

- Snaphost, Freq vs. W.



:pencil2: **Revision**

- Compact, Flag: `-auto-compaction`
- Defrag



:pencil2: **Options**

- multi-DC, `–heartbeat-interval` (100ms) → 0.55/1.5 * peer RTT.


- `–election-timeout` (1000ms) → 10 * peer RTT.



:pencil2: **Backup & Recover**

- wal & snap, `–snapshot-count` (100,000) & `–max-snapshots` (5)
- Freq
- Increment (Snapshot + Events within Snapshot duration) ≈ Redis RDB + AOF.
