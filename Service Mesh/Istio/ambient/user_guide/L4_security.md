**Only L4 Policy enforcement by ztunnel**

AuthN: native mTLS, default is PERMISSIVE, can enable STRICT but DISABLE not allowed!

AuthZ: policy rules can contain [source](https://istio.io/latest/docs/reference/config/security/authorization-policy/#Source) (`from`), [operation](https://istio.io/latest/docs/reference/config/security/authorization-policy/#Operation) (`to`), and [condition](https://istio.io/latest/docs/reference/config/security/authorization-policy/#Condition) (`when`) clauses.

```yaml
# L4
# can be used in both sidecar & ambient mode
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: allow-sleep-to-httpbin
spec:
 selector:
   matchLabels:
     app: httpbin
 action: ALLOW
 rules:
 - from:
   - source:
       principals:
       - cluster.local/ns/ambient-demo/sa/sleep
```

```yaml
# L7 will fail safe by becoming a DENY policy.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: allow-sleep-to-httpbin
spec:
 selector:
   matchLabels:
     app: httpbin
 action: ALLOW
 rules:
 - from:
   - source:
       principals:
       - cluster.local/ns/ambient-demo/sa/sleep
   to:
   - operation:
       methods: ["GET"]
EOF

```





