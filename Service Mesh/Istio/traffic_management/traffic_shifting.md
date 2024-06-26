### [Traffic Shifting](https://istio.io/latest/docs/tasks/traffic-management/traffic-shifting/)

#### HTTP

A common use case is to migrate traffic gradually from an older version of a microservice to a new one.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews    # 50% to v1
        subset: v1
      weight: 50 
    - destination:
        host: reviews    # 50% to v3
        subset: v3
      weight: 50
```



**Blue/Green**: <u>reduces downtime and risk</u> by running two identical production environments, referred to as Blue (current) and Green (live), only one is live at any given time.



![What is Blue-Green Deployment](https://candost.blog/images/content/posts/blue-green-deployment/BlueGreenDeployment5.png)



**Canary Release**: <u>reduce the risk of introducing a new software version</u> by gradually rolling it out to a subset of users (low percentage) before making it available to the entire infrastructure or user base.



![img](https://cdn.sanity.io/images/rhzn5s2f/production/f5224ec3a4c0de15ed8b41413e982c88248863cb-780x330.webp?w=1920&fit=max&auto=format)



**A/B Test**: comparing two versions of a webpage or app against each other to <u>determine which one performs better</u>. Target is to shifting the entire traffic from one to another.



![A/B-testning - Optimizely](https://www.optimizely.com/contentassets/08726e145f1b4743a0ba2f30c0447b76/ab-testing.png)

#### [TCP](https://istio.io/latest/docs/tasks/traffic-management/tcp-traffic-shifting/)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tcp-echo
spec:
  hosts:
  - "*"
  gateways:
  - tcp-echo-gateway
  tcp:
  - match:
    - port: 31400
    route:
    - destination:
        host: tcp-echo   # 80% to v1
        port:
          number: 9000
        subset: v1
      weight: 80
    - destination:
        host: tcp-echo   # 20% to v2
        port:
          number: 9000
        subset: v2
      weight: 20
```

