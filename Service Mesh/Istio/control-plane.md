The **control plane** manages and configures the proxies to route traffic.



![The overall architecture of an Istio-based application.](control-plane.assets/arch.svg)



### Pilot

- It is responsible for converting high-level conf & SD info → specific conf objects that Envoy can read such as:
  - **SR(egistration) & SD(iscovery)**
    - Watches svc/ep (& health status) & CRD (ServiceEntries) → internal svc reg.
  - **Topo Gen**
    - Dynamic Repr with policies enforcement such as LB & Fail-over.
  - **Endpoint Selection**
    - Forward req based on decisions like path/header. → healthy ep.
  - **HC**
    - Relies on (updates from) K8s.
  - **Dynamic Updates**
    - Watch & distribute.



### Flow

productpage → reviews where

- Routes by match → `VirtualServices` → ++entry in routes
- Subsets         → `DestinationRule` → ++entries in clusters

```bash
$ POD_NAME=$(kubectl get pod -l app=productpage -n default -o jsonpath='{.items[0].metadata.name}')

# listener
$ istioctl proxy-config listeners $POD_NAME

# clusters
$ istioctl proxy-config clusters $POD_NAME

# routes 
$ istioctl proxy-config routes $POD_NAME
```





![img](https://miro.medium.com/v2/resize:fit:1380/1*HTStO76UNJwYVr9PY-s6dQ.png)