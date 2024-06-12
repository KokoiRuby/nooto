:confused: What is [etcd](https://etcd.io/)?

- A strongly consistent & reliable (Raft), distributed key-value store.

  

:confused: **Features?**

- RW over simple HTTP(S)

- KV storage

- Key TTL/Lease for watch & svc discovery

- Fast 1000 W/s & 2000 R/s

  

:confused: **[Quickstart](https://github.com/KokoiRuby/docker/tree/main/etcd)?**



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

  

:confused: **OAM?**

- 