### [Fault Injection](https://istio.io/latest/docs/tasks/traffic-management/fault-injection/)

Fault injection policy to apply on HTTP traffic at the **client** side. **To emulate errors in production & verify if system is resilient to**.

- [delay](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPFaultInjection-Delay): emulating network issue, overloaded upstream...
- [abort](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPFaultInjection-Abort): emulating upstream is faulty (4xx, 5xx)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:               # 7s delay on forward all requests to ratings for jason user.
      delay:
        percentage:
          value: 100.0
        fixedDelay: 7s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:              # return 500 for all requests to ratings for jason user.
      abort:
        percentage:
          value: 100.0
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

