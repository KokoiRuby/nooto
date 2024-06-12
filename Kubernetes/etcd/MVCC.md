:confused: **Concurrency Control?**

- Pessimistic: lock before op, assuming conflict is more likely to occur, cost: rollback > lock

- Optimisitc: lock after op, assuming conflict is less likely to occur, cost: lock > rollback

- MVCC: W ++ rev, R by rev → avoid conflict.

  ```bash
  $ etcdctl put hello world1
  $ etcdctl get hello -w json | grep revision   # rev: 1
  $ etcdctl put hello world2
  $ etcdctl get hello -w json | grep revision   # rev: 2
  $ etcdctl get hello
  $ etcdctl get hello --rev 1
  
  $ etcdctl delete hello
  $ etcdctl get hello --rev 1  # world1
  $ etcdctl get hello --rev 2  # world2
  ```

  

:confused: **Why etcd choose MVCC?**

- It keeps **meta** for shared conf & svc discovery.
- **R more** W less.



:confused: **v2 → v3?**

- v2 pure in-mem, Stop-the-World, RW lock block to each other.
- v3 ++ on-disk, W lock will block R lock, support currenct R.



:confused: **Arch?**

- ReadTxn → range, WriteTxn → put/delete
- **In-mem treeIndex, Btree: key → rev**
- **On-disk boltdb, B+tree: rev → v**



![img](https://img2023.cnblogs.com/blog/1229382/202306/1229382-20230601151453891-1269317625.png)



![mvcc data model](https://etcd.io/docs/v3.5/learning/img/data-model-figure-01.png)