### [Mirroring](https://istio.io/latest/docs/tasks/traffic-management/mirroring/)

AKA shadowing, a powerful concept that allows feature teams to bring changes to production with as little risk as possible. It sends a copy of live traffic to a mirrored service ← Fire & Forget = Drop any responses.

UC: live troubleshooting, productin stress.

```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
    - httpbin
  http:
  - route:
    - destination:
        host: httpbin
        subset: v1
      weight: 100
    mirror:
      host: httpbin
      subset: v2
    mirrorPercentage:
      value: 100.0
EOF
```

