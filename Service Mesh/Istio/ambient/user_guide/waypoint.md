An optional **deployment of the Envoy-based proxy** to enforce L7 features. Unlike sidecar, waypoint can be  installed, upgraded and scaled **independently** from applications.

A waypoint, or set of waypoints, can be **shared** between applications with a similar security boundary.



:confused: **Do u need a waypoint proxy?**

- **Traffic management**: HTTP routing & load balancing, circuit breaking, rate limiting, fault injection, retries, timeouts
- **Security**: Rich authorization policies based on L7 primitives such as request type or HTTP header
- **Observability**: HTTP metrics, access logging, tracing



:confused: **How to deploy?**

```bash
# ns must enable ambient mode
$ kubectl get ns -L istio.io/dataplane-mode
```

```bash
# gen spec & apply
$ istioctl experimental waypoint generate --for service -n default
$ istioctl experimental waypoint apply -n default

# or
$ kubectl apply -f - <<EOF
kind: Gateway
metadata:
  labels:
    # indicating the waypoint can process traffic for services, which is the default.
    istio.io/waypoint-for: service
  name: waypoint
  namespace: default
spec:
  gatewayClassName: istio-waypoint
  listeners:
  - name: mesh
    port: 15008
    protocol: HBONE
EOF
```

Traffic types:

| `waypoint-for` value | Traffic type                      |
| -------------------- | --------------------------------- |
| `service`            | Kubernetes services               |
| `workload`           | Pod or VM IPs                     |
| `all`                | Both service and workload traffic |
| `none`               | No traffic (useful for testing)   |



:confused: **How to use?**

Only labeled with a value of waypoint name can waypoint be used.

```bash
$ istioctl experimental waypoint apply -n default --enroll-namespace
# or
$ kubectl label ns default istio.io/use-waypoint=waypoint
```

After enrollment, **any requests from any pods using the ambient data plane mode, to any service running in that ns**, will be routed through the waypoint for L7 processing and policy enforcement.

```bash
# enroll for svc
$ istioctl experimental waypoint apply -n default --name reviews-svc-waypoint
$ kubectl label service reviews istio.io/use-waypoint=reviews-svc-waypoint

# encroll for pod
$ istioctl experimental waypoint apply -n default --name reviews-v2-pod-waypoint --for workload
$ kubectl label pod -l version=v2,app=reviews istio.io/use-waypoint=reviews-v2-pod-waypoint
```

