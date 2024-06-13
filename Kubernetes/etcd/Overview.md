:confused: What is [etcd](https://etcd.io/)?

- A strongly consistent & reliable (Raft), distributed key-value store.

  

:confused: **Features?**

- RW over simple HTTP(S)

- KV storage

- Key TTL/Lease for watch & svc discovery

- Fast 1000 W/s & 2000 R/s




:confused: **CAP?**

- **C**onsistency, **A**vailability, (Network) **P**artition tolerance
- P must be secured. CP or AP only.



:confused: **Arch?**



![img](https://upload-images.jianshu.io/upload_images/9243349-d3a5a1c4b81aa3c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

:confused: **Client [Arch](https://etcd.io/docs/v3.5/learning/design-client/)?**

| Term                 | Desc                                                         |
| -------------------- | ------------------------------------------------------------ |
| Balancer             | Impl retry & failover.                                       |
| Endpoints            | A list of etcd servers endpoints.                            |
| Pinned endpoint      | **RR for every req since v3.4.**                             |
| Client Connection    | gRPC btw.                                                    |
| Sub Connection       | Per endpoint.                                                |
| Transient disconnect | [`code Unavailable`](https://godoc.org/google.golang.org/grpc/codes#Code). |



![client-balancer-figure-09.png](https://etcd.io/docs/v3.5/learning/img/client-balancer-figure-09.png)



:confused: **[Quickstart](https://github.com/KokoiRuby/docker/tree/main/etcd_new)?**



:confused: **MVCC?** Multi-Version Concurrent Control

- Optimisic Lock
  - W → append new ver. (instead of overwrite)
  - R ← specific ver.
  - D → tombstone (instead of immediate removal)
  
- All K in flat namespace, ordered by dict

- **Don't forget etcd itself is a State Machine, rev is to describe kv state, each state if a collection of k updates.**

  

:confused: **Storage?**

- in-mem Btree

- on-disk B+tree bblot (major/sub/type)

  

:confused: **API?**

- KV
  - Range
  - Put
  - DeleteRange
  - Txn
  
- Watch

- Lease (to check if client is alived)

  

:confused: **gRPC header?**

- cluster_id 

- member_id

- revision

- raft_term

