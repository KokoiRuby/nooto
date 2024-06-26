### [Locality Load Balancing](https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/)

A *locality* defines the geo location of a workload instance within your mesh.

- Region: large geo area, such as us-east/cn-west. [`topology.kubernetes.io/region`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesioregion) 
- Zone: A set of compute resources within a region, such as us-east-1/cn-west-2. [`topology.kubernetes.io/zone`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
- Sub-zone: division of zone. [`topology.istio.io/subzone`](https://istio.io/latest/docs/reference/config/labels/#:~:text=topology.istio.io/subzone) ← Istio++



![Regions](https://k21academy.com/wp-content/uploads/2023/03/regions.png)



#### [Failure](https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/failover/)

Mesh → region1(zone1 + znoe2), region2, region3

![Locality failover sequence](https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/failover/sequence.svg)

| Priority | Locality        | Details                                                      |
| -------- | --------------- | ------------------------------------------------------------ |
| 0        | `region1.zone1` | Region, zone, and sub-zone all match.                        |
| 1        | None            | Since this task doesn’t use sub-zones, there are no matches for a different sub-zone. |
| 2        | `region1.zone2` | Different zone within the same region.                       |
| 3        | `region2.zone3` | No match, however failover is defined for `region1`->`region2`. |
| 4        | `region3.zone4` | No match and no failover defined for `region1`->`region3`.   |

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: helloworld
spec:
  host: helloworld.sample.svc.cluster.local
  trafficPolicy:
    connectionPool:
      http:
        maxRequestsPerConnection: 1
    loadBalancer:
      simple: ROUND_ROBIN
      localityLbSetting:
        enabled: true
        failover:
          - from: region1
            to: region2
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 1m
```

### [Weighted distribution](https://istio.io/latest/docs/tasks/traffic-management/locality-load-balancing/distribute/)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: helloworld
spec:
  host: helloworld.sample.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      localityLbSetting:
        enabled: true
        distribute:
        - from: region1/zone1/*
          to:
            "region1/zone1/*": 70
            "region1/zone2/*": 20
            "region3/zone4/*": 10
    outlierDetection:
      consecutive5xxErrors: 100
      interval: 1s
      baseEjectionTime: 1m
```

