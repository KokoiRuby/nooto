#### [Traffic redirection](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/)

Data plane functionality that intercepts traffic sent to and from ambient-enabled workloads, routing it through the ztunnel node proxies

- `istio-cni` pod responds to CNI events such as pod creation/deletion & write iptables rules into pod network ns.
- `istio-cni` node agent then informs the `ztunnel` proxy, over a Unix domain socket, that it should establish local proxy listening ports inside the pod’s network namespace. (15008, 15006, 15001)
- The node-local `ztunnel` internally spins up a new proxy instance and listen port set, dedicated to the newly-added pod.
- Once the in-pod redirect rules are in place and the `ztunnel` has established the listen ports, the pod is added in the mesh and traffic begins flowing through the node-local ztunnel.
- **Traffic to and from pods in the mesh will be fully encrypted with mTLS by default.**

```bash
$ kubectl get ds -n istio-system
```



![pod added to the ambient mesh flow](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/pod-added-to-ambient.svg)



![HBONE traffic flows between pods in the ambient mesh](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/traffic-flows-between-pods-in-ambient.svg)

### L4

`ztunnel` as base

- Per-node daemon.
- L4
  - Traffic Mgmt: TCP routing.
  - Security: mTLS, Simple AuthZ.
  - Observability: TCP metrics & logging.
- [HBONE](https://istio.io/latest/docs/ambient/architecture/hbone/) tunneling protocol.

| pod     | ip          | pid  | worker                |
| ------- | ----------- | ---- | --------------------- |
| sleep   | 10.244.2.4  | 2425 | istio-sidecar-worker2 |
| product | 10.244.2.14 | 3056 | istio-sidecar-worker2 |
| ztunnel | 10.244.2.11 | 2691 | istio-sidecar-worker2 |



```bash
# sleep (10.244.2.4) → productpage (10.244.2.14) same node
$ kubectl exec deploy/sleep -- curl -s http://productpage:9080/productpage | grep -o "<title>.*</title>"
```

流量被同节点的 tunnel 以 TPROXY（透明代理 = 保留原始目的地址）方式拦截转发到当前节点的 ztunnel

```bash
# on worker
$ crictl ps | egrep "sleep|product|ztun"
$ crictl inspect -o go-template --template '{{.info.pid}}' <container_id>
$ nsenter -t <pid> -n iptables-save
```

sleep 容器内 iptables rule 由 `istio-cni` 注入，目标地址非本机的 TCP 重定向到端口 15001

```bash
-A OUTPUT -j ISTIO_OUTPUT
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -p tcp -m mark ! --mark 0x539/0xfff -j REDIRECT --to-ports 15001
```



