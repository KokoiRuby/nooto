### Host

- Host provides runtime for proc ← **USE**

  

- Out-band：not via business network by IMPI | SNMP → HW ← IDC O&M (Internet Data Center) 

- In-band：via business network，Agent-in-OS ← Cloudnative O&M

  - [Categraf](https://github.com/flashcatcloud/categraf) | [Telegra](https://github.com/influxdata/telegraf) | Grafana-agen | Datadog-agent | Node-exporter

  

- Prometheus accepts RemoteWrite

  - API /api/v1/write

  - `--enable-feature`=remote-write-receiver

    

- Metrics

  - cpu | mem | disk | diskio | net | socket | proc | vmstat | ntp | exec

### Network

- **Link** 
  - Connectivity: ICMP | TCP | HTTP → liveness.
  - traffic + content → watermark & qos.
- **Device**
  - ICMP → liveness.
  - SNMP to pull，UDP 161.
    - Scaper → (OID) → SNMP agent on Device
  - SNMP Trap for alarm.
    - SNMP agent → SNMP Manager

### Component

#### MySQL

- Duration (Successfulk vs. Failed)
  - Instrumentation → delta(before & after).
  - `Slow_queries` in global status.
  - Schema: performance & sys.
- TPS
  - R/W `Com_*` in global status.
  - `Questions` (# of statement from client → MySQL) vs. `Queries` (++ statement still in-process)
- Error
  - Instrumentation → marked once connection failed.
  - `Connection_errors_max_connections` & `Aborted_connects` in global status.
  - Schema: performance
- Saturation “Queue”
  - Threads_connected / max_connections
  - `%buffer%` in global status → cache Table & Index.
    - `Innodb_buffer_pool_pages_total`
    - `Innodb_buffer_pool_pages_free`
    - `Innodb_buffer_pool_read_requests`
    - `Innodb_buffer_pool_reads`

#### Redis

- Duration (Successfulk vs. Failed)
  - Instrumentation delta(before & after).
  - `redis-cli --latency`
  - `SLOWLOG get`
- TPS
  - info all
    - `instantaneous_*`
    - `keyspace_[hits|misses]`
- Error
  - info all
    - `rejected_connections`
- Saturation “Queue”
  - info memory
    - `used_memory_rss_human` (OS Allocation) / `used_memory_human` (used by Redis)
      - Over  1 = Fragmentation → active defrag
      - Below 1 = Swap
  - evicted_keys
    - strategy
- Misc
  - `rdb_changes_since_last_save`
  - `master_link_down_since_seconds`

#### Kafka (Broker)

- Host: CPU & IO & FD ← **USE**
- JVM: JMX - GC
- Kakfa: JMX
  - MBean：broker
    - kafka.controller:type=KafkaController,name=ActiveControllerCount
    - kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions
    - kafka.controller:type=KafkaController,name=OfflinePartitionsCount
    - kafka.log:type=LogManager,name=OfflineLogDirectoryCount
    - kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec
    - kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec
    - kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
    - kafka.server:type=ReplicaManager,name=PartitionCount
    - kafka.server:type=ReplicaManager,name=LeaderCount
  - kafka_consumer_lag_millis

#### Elasticsearch

- Throughput & Latency

- Endpoint

  - /_nodes/stats
    - indices
      - indexing
      - search
      - fetch
      - merges

  - /_cluster/health

#### Kubernetes

- All components expose /metrics；

- For public cloud, control plane is usually managed by provider, users care **data plane** more.

  

- Data Plane

  - General
    - `go_*` 
    - `rest_client_request_duration_seconds`
    - `rest_client_requests_total`
  - kube-proxy (10249，token no needed)
    - `kubeproxy_sync_proxy_ruiles_*`
  - kubelet (10250，token needed)
    - `kubelet_<runtime>`
    - `kubelet_network_plugin_*`
  - "workload"
    - cpu
      - `container_cpu_usage_seconds_total`
      - `container_cpu_throttled_seconds_total`
      - `container_cpu_cfs_periods_total`
      - `container_spec_cpu_quota`
      - `container_spec_cpu_period`
    - mem
      - `container_memory_working_set_bytes`
      - `container_spec_memory_limit_bytes`
      - `container_spec_memory_limit_bytes`
    - net
      - `container_network_transmit_bytes_total`
      - `container_network_receive_bytes_total`
    - diskio
      - `container_fs_reads_bytes_total`
      - `container_fs_writes_bytes_total`

- Control Plane

  - API Server (6443)
    - `apiserver_request_total`
    - `apiserver_request_duration_seconds`
    - `apiserver_current_inflight_requests`
  - controller-manager
    - `workqueue_adds_total`
    - `workqueue_depth`
    - `workqueue_queue_duration_seconds`
    - `workqueue_work_duration_seconds`
    - `workqueue_retries_total`
  - schduler
    - `leader_election_master_status`
    - `scheduler_queue_incoming_pods_total`
    - `scheduler_pending_pods`
    - `scheduler_pod_scheduling_attempts`
    - `scheduler_framework_extension_point_duration_seconds`
    - `scheduler_schedule_attempts_total`
  - etcd
    - `etcd_server_has_leader`
    - `etcd_server_leader_changes_seen_total`
    - `etcd_server_proposals_failed_total`
    - `tcd_disk_backend_commit_duration_seconds`
    - `etcd_disk_wal_fsync_duration_seconds`
  - kube-state-metrics
    - `kube_node_status_condition`
    - `kube_pod_container_status_last_terminated_reason`
    - `kube_pod_container_status_waiting_reason`
    - `kube_pod_container_status_restarts_total`
    - `kube_deployment_spec_replicas`
    - `kube_deployment_status_replicas_available`