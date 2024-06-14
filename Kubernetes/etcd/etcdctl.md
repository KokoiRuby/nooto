[etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl) is a command line client for [etcd](https://github.com/coreos/etcd). Make sure u've [setup](https://github.com/KokoiRuby/docker/tree/main/etcd_new) a 3-node cluster first.

[etcdutl](https://github.com/etcd-io/etcd/tree/main/etcdutl) is a command line **administration** utility for [etcd](https://github.com/coreos/etcd).

```bash
# global flags if tls
ETCDCTL_DIAL_TIMEOUT=10s
ETCDCTL_CACERT=/tmp/ca.pem
ETCDCTL_CERT=/tmp/cert.pem
ETCDCTL_KEY=/tmp/key.pem

# endpoint flag
--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379
```

## OAM

```bash
# endpoint
$ etcdctl endpoint health
$ etcdctl endpoint status
$ etcdctl endpoint status -w [json|table]
$ etcdctl endpoint hashkv
$ etcdctl endpoint hashkv -w [json|table]


# membership
$ etcdctl member list
$ etcdctl member list -w [json|table]
```

### [Add/Remove Member](https://etcd.io/docs/v3.6/tutorials/how-to-deal-with-membership/)

```bash
# remove
$ etcdctl member remove <id> \
	--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379

# chk
$ etcdctl member list \
	--endpoints http://etcd0:2379,http://etcd1:2379

# add 
$ etcdctl --endpoints=http://etcd0:2379,http://etcd1:2379 \
	member add etcd2 \
	--learner \
	--peer-urls=http://etcd2:2380

# ETCD_NAME="etcd2"
# ETCD_INITIAL_CLUSTER="etcd2=http://etcd2:2380,etcd0=http://etcd0:2380,etcd1=http://etcd1:2380"
# ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd2:2380"
# ETCD_INITIAL_CLUSTER_STATE="existing"


# start new member, modify compose.yaml
# ETCD_INITIAL_CLUSTER_STATE=new → existing
  etcd2:
    ...
    environment:
      ...
      - ETCD_INITIAL_CLUSTER_STATE=existing

# chk
$ etcdctl member list \
	--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379
```

### Leadership

```bash
# locate Follower to transfer
$ etcdctl endpoint status -w table \
	--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379

$ etcdctl move-leader ed7f1c7ec3041676 \
	--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379

# chk
$ etcdctl endpoint status -w table \
	--endpoints http://etcd0:2379,http://etcd1:2379,http://etcd2:2379
```

```bash
# participate name election
$ etcdctl elect myelection p1
$ etcdctl elect myelection p2
$ etcdctl elect myelection p3
```

### [Debug large db size issue](https://etcd.io/blog/2023/how_to_debug_large_db_size_issue/)

`--quota-backend-bytes`: max number of bytes the etcd db file may consume, default 2GB, suggested max is 8GB. Alarm & Err msg when breach.

```bash
# log: etcdserver: mvcc: database space exceeded
# alarm
$ etcdctl endpoint status
$ etcdctl endpoint health
$ etcdctl alarm list
# need to clear alarm after space is freed
$ etcdctl alarm disarm

# chk db size, "db" file
$ ls -lrt /path-2-db-data-dir/member/snap/

# metrics
etcd_server_quota_backend_bytes
etcd_mvcc_db_total_size_in_bytes
etcd_mvcc_db_total_size_in_use_in_bytes
```

The compaction is the ONLY way to purge key history in MVCC, nee also defrag to reclaim the free space.

- `--auto-compaction-retention` (hour)

```bash
$ etcdctl compact <rev>
# etcdserver: mvcc: required revision has been compacted
$ etcdctl get --rev=<rev-1> somekey

$ etcdctl defrag
$ etcdctl defrag --cluster
# offline
$ etcdctl defrag --data-dir <path-to-etcd-data-dir>
```

### Backup & Restore

```bash
$ etcdctl snapshot save backup.db
$ etcdutl snapshot status backup.db --write-out=[json|table]

# to be removed in v3.6, use "etcdutl" instead
$ etcdctl snapshot restore <snapshot-file-name> \
  --data-dir <data-directory> \
  --name <member-name>
  
# since v3.6
$ etcdutl snapshot restore snapshot.db \
	--initial-cluster-token token \
	--initial-advertise-peer-urls http://etcd0:2380 \
	--name etcd0 \
	--initial-cluster 'etcd0=http://etcd0:2380,etcd1=http://etcd1:2380,etcd2=http://etcd2:2380'
```

## UC

- K/V store for conf

  ```bash
  $ etcdctl put /app/conf/max_conn 100
  $ etcdctl txn --interactive
  ```

  ```bash
  $ etcdctl get /app/conf/max_conn
  $ etcdctl get --prefix /app/conf
  $ etcdctl get --prefix /app/conf --limit=10
  $ etcdctl get --prefix /app/conf --keys-only
  $ etcdctl get --prefix /app/conf --keys-only --debug
  ```

  ```bash
  $ etcdctl watch /app/conf/max_conn
  $ etcdctl watch --prefix /app/conf/
  ```

  ```bash
  $ etcdctl del /app/conf/max_conn
  $ etcdctl del --prefix /app/conf/
  ```

- Service Discovery

  ```bash
  # reg
  $ etcdctl put /svc/app1/server1 '{"host": "192.168.1.100", "port": 8080}'
  # discover
  $ etcdctl get /services --prefix
  ```

- Distributed Lock 

  ```bash
  $ etcdctl lock mutex
  $ etcdctl lock mutex --ttl=10
  $ etcdctl unlock mutex
  ```

- Lease

  ```bash
  $ etcdctl lease grant 60
  $ etcdctl put --lease=<id> /key value
  # refresh
  $ etcdctl lease keep-alive <lease-id>
  # revoke
  $ etcdctl lease revoke <id>
  ```

- Limit

  ```bash
  $ etcdctl get --prefix --limit=10
  ```

  

