### Long leaving files

```bash
$ ./member/snap/db
```

- **bbolt [b+tree](https://en.wikipedia.org/wiki/B%2B_tree)** that stores all the applied data, membership authorization information & metadata

- [etcd-dump-db](https://github.com/etcd-io/etcd/tree/main/tools/etcd-dump-db) :construction_worker:

  

```bash
# term-index
$ ./member/snap/0000000000000002-0000000000049425.snap
$ ./member/snap/0000000000000002-0000000000061ace.snap
```

- Periodic **snapshots of legacy v2 store**, as of etcd v3, the content is redundant to the content of /snap/db files.

- `--max-snapshots`

- Triggered when:

  - every (approximately) `–snapshotCount` applied proposals.
  - Raft requests the replica to restore from the snapshot.

  

```bash
# index
$ /member/snap/000000000007a178.snap.db
```

- A complete **bbolt snapshot downloaded** from the etcd leader if the replica was lagging too much, scenarios:

  - Recover from snapshot, will be deleted after recovery is over.
  - Server statup and found newer index.

- `--max-snapshots`

- :warning: Risk of disk space exhaustion.

  

```bash
# startIdx-endIdx
$ ./member/wal/000000000000000f-00000000000b38c7.wal
```

- **Raft’s Write Ahead Logs**, containing recent transactions accepted by Raft, periodic snapshots or CRC records.
  - First the leader stores the proposal in its log and then (concurrently) replicates to followers.
  - Each follower persists the proposal in its WAL before confirming back replication to the leader.
- `--max-wals`
- [etcd-dump-db](https://github.com/etcd-io/etcd/tree/main/tools/etcd-dump-db) :construction_worker:



```bash
$ ./member/wal/0.tmp (or .../1.tmp)
```

- **Preallocated space** for the next WAL file to avoid Raft being stacuk by a lack of WAL logs capacity.











