### [Request Timeout](https://istio.io/latest/docs/tasks/traffic-management/request-timeouts/)

Timeout is to **avoid requests waiting indefinitely** for a response, preventing request backlog and resource exhaustion. By default, the request timeout is disabled.

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
        host: reviews
        subset: v2
    timeout: 1s      # timeout
```