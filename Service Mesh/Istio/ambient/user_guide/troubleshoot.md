### ztunnel

ztunnel gets conf and discovery info from the istiod via xDS.

```bash
# list all workloads that ztunnel is currently tracking
$ istioctl x ztunnel-config

# view secrets holding TLS cert that ztunnel received from istiod for mTLS.
$ istioctl x ztunnel-config certificates "$ZTUNNEL".istio-system

# conf
$ istioctl x ztunnel-config all -o json
$ kubectl debug -it $ZTUNNEL -n istio-system --image=curlimages/curl \
	-- curl localhost:15000/config_dump
```

```json
"/10.244.1.4": {
      "canonicalName": "notsleep",
      "canonicalRevision": "latest",
      "clusterId": "Kubernetes",
      "name": "notsleep-57bfbb845f-rw7kp",
      "namespace": "default",
      "node": "istio-sidecar-worker",
      "protocol": "TCP",
      "serviceAccount": "notsleep",
      "status": "Healthy",
      "trustDomain": "cluster.local",
      "uid": "Kubernetes//Pod/default/notsleep-57bfbb845f-rw7kp",
      "workloadIps": [
        "10.244.1.4"
      ],
      "workloadName": "notsleep",
      "workloadType": "deployment"
    },
```

Traffic logs

```bash
$ kubectl -n istio-system logs -l app=ztunnel | grep -E "inbound|outbound"
$ kubectl -n istio-system logs -f "$ZTUNNEL"
```

LB: internally fixed L4 RR.

```bash
$ kubectl -n istio-system logs -l app=ztunnel | grep -E "outbound"
```

### waypints

```bash
# check if any validation issue before applying L7 conf
$ istioctl analyze

# which waypoint is implementing L7 conf for svc/pod
$ istioctl x ztunnel-config service
$ istioctl x ztunnel-config workload

# proxy status
$ istioctl proxy-status
```

```bash
# envoy access log
$ kubectl logs deploy/waypoint

# enable debug for waypoint
$ istioctl pc log deploy/waypoint --level debug
```

```bash
# envoy conf - listener → clusters → route → endpoint
$ istioctl proxy-config all deploy/waypoint
```

