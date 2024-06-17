### [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)

```bash
$ kubectl label nodes <node> key=value
$ kubectl label nodes <node> key-
```

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
  # ...
  nodeSelector:
    key: value
```

### [Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

```bash
$ kubectl label nodes <node> key=value
$ kubectl label nodes <node> key-
```

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: key1
            operator: In
            values:
            - value1_1
            - value1_2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: key2
            operator: In
            values:
            - value2
# ...
```

- **topologyKey**: control scope of [anti]affinity → node label
  - kubernetes.io/hostname
  - failure-domain.beta.kubernetes.io/zone
  - failure-domain.beta.kubernetes.io/region

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: key2
              operator: In
              values:
              - value2
          topologyKey: topology.kubernetes.io/zone
# ...

```

### [nodeName](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)

- kube-scheduler will ignore if specify explicitly.

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
# ...
  nodeName: kube-01
```

### [Pod topology spread constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)

- Make sure pod spread among differernt domain, HA ↑
- Pods spread over certain domain, delay ↓
- Re-schedule when node in topo is down.

```yaml
---
apiVersion: v1
kind: Pod
# ...
spec:
  # Configure a topology spread constraint
  topologySpreadConstraints:
    - maxSkew: <integer>
      minDomains: <integer> # optional
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
      matchLabelKeys: <list>             # optional; beta since v1.27
      nodeAffinityPolicy: [Honor|Ignore] # optional; beta since v1.26
      nodeTaintsPolicy: [Honor|Ignore]   # optional; beta since v1.26
  # other Pod fields go here
```

### [Taint & Tolerance](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

```bash
# taint & untaint node
# effect: NoExecute (tolerationSeconds), PreferNoSchedule, NoExecute
kubectl taint node <node> {key}={value}:{effect}
kubectl taint node <node> {key}-

# cordon
# ++taint
# node.kubernetes.io/unschedulable:NoSchedule
kubectl cordon <node>
kubectl uncordon <node>

# drain - stop scheduling to current node, terminate pods gracefully & schedcule them to other nodes.
# drain will trigger cordon first
kubectl drain <node>
kubectl drain <node> --ignore-daemonsets
```

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
# ...
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"

```

### **[PriorityClass](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass)**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "This priority class is for high priority pods."
---
apiVersion: v1
kind: Pod
# ...
spec:
  priorityClassName: high-priority
# ...
```

