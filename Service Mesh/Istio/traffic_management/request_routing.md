### [Request Routing](https://istio.io/latest/docs/tasks/traffic-management/request-routing/)

**[Virtual Service](https://istio.io/latest/docs/reference/config/networking/virtual-service/)**：defines a set of traffic **routing rules** to apply when a host is addressed. If the traffic is matched, then it is sent to a named destination service.

- hosts: The destination hosts to which traffic is being sent. → HTTP Request Header → `Host`
- [http](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRoute)
  - match.[uri|header|sourceLabels|rewrite]
  - route (++weight), delegate
  - timeout, retry, fault, mirrors

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: reviews-route
  namespace: foo
spec:
  hosts:
  - reviews # interpreted as reviews.foo.svc.cluster.local
  http:
  - match:
    - uri:
        prefix: "/wpcatalog"
    - uri:
        prefix: "/consumercatalog"
    rewrite:
      uri: "/newcatalog"
    route:
    - destination:
        host: reviews # interpreted as reviews.foo.svc.cluster.local
        subset: v2
  - route:
    - destination:
        host: reviews # interpreted as reviews.foo.svc.cluster.local
        subset: v1

```

[Destination Rule](https://istio.io/latest/docs/reference/config/networking/destination-rule/): defines **policies** that apply to traffic intended for a service **after routing** has occurred. Version-based subsets.

- hosts: The name of a service from the service registry.
- workloadSelector: label-based
- [trafficPolicy](https://istio.io/latest/docs/reference/config/networking/destination-rule/#TrafficPolicy): circuit-breaker, loadbalancer, tls (to upstream)

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: reviews-destination
  namespace: foo
spec:
  host: reviews # interpreted as reviews.foo.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```