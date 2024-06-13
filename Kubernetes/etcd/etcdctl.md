[etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl) is a command line client for [etcd](https://github.com/coreos/etcd). Make sure u've [setup](https://github.com/KokoiRuby/docker/tree/main/etcd_new) a 3-node cluster first.

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
$ etcdctl endpoint status
$ etcdctl endpoint status -w [json|table]

# membership
$ etcdctl member list
$ etcdctl member list -w [json|table]
```

### [Add/Remove Member](Add/Remove Member)

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

### Transfer Leadership

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

### [Debug large db size issue](https://etcd.io/blog/2023/how_to_debug_large_db_size_issue/)



## [Proxy](https://etcd.io/docs/v3.5/op-guide/grpc_proxy/)



## UC

- K/V store for conf

  ```bash
  $ etcdctl put /app/conf/max_conn 100
  $ etcdctl get /app/conf/max_conn
  $ etcdctl get --prefix /app/conf
  $ etcdctl get --prefix /app/conf --limit=10
  $ etcdctl watch /app/conf/max_conn
  $ etcdctl del /app/conf/max_conn
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
  $ etcdctl lock mylock
  $ etcdctl lock mylock --ttl=10
  $ etcdctl unlock mylock
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

  

