### Helm

```bash
$ helm repo add istio https://istio-release.storage.googleapis.com/charts
$ helm repo update
```

```bash
# crd & cluster role
$ helm install istio-base istio/base -n istio-system --create-namespace --wait
# CNI plugin
$ helm install istio-cni istio/cni -n istio-system --set profile=ambient --wait
# istiod as control plane
$ helm install istiod istio/istiod -n istio-system --set profile=ambient --wait
# daemonset node-agent
$ helm install ztunnel istio/ztunnel -n istio-system --wait
```

```bash
# verify
$ helm ls -n istio-system
$ kubectl get pods -n istio-system | egrep "istio|ztun"
```

```bash
$ helm delete istio-cni -n istio-system
$ helm delete ztunnel -n istio-system
$ helm delete istiod -n istio-system
$ helm delete istio-base -n istio-system

# cleanup
$ kubectl delete namespace istio-system
```

### istioctl

```bash
$ istioctl install --set profile=ambient --skip-confirmation

# cleanup
$ istioctl uninstall -y --purge
```

### [Getting Started](https://istio.io/latest/docs/ambient/getting-started/)


```bash
# make sure istio-injection is disabled
$ kubectl label namespace default istio-injection-
```

```bash
# ++ to mesh
$ kubectl label namespace default istio.io/dataplane-mode=ambient

$ export GATEWAY_HOST=bookinfo-gateway-istio.default
$ export GATEWAY_SERVICE_ACCOUNT=ns/default/sa/bookinfo-gateway-istio

# verify from in-cluster
$ kubectl exec deploy/sleep -- curl -s "http://$GATEWAY_HOST/productpage" | grep -o "<title>.*</title>"
$ kubectl exec deploy/sleep -- curl -s http://productpage:9080/ | grep -o "<title>.*</title>"
$ kubectl exec deploy/notsleep -- curl -s http://productpage:9080/ | grep -o "<title>.*</title>"
```

```yaml
# k8s Gateway
spec:
  gatewayClassName: istio
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: http
    port: 80
    protocol: HTTP
status:
  addresses:
  - type: Hostname
    # name-gatewayClassName.ns.domainName
    value: bookinfo-gateway-istio.default.svc.cluster.local
---
# k8s HTTPRoute
spec:
  parentRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: bookinfo-gateway
  rules:
  - backendRefs:
    - group: ""
      kind: Service
      name: productpage
      port: 9080
      weight: 1
    matches:
    - path:
        type: Exact
        value: /productpage
    - path:
        type: PathPrefix
        value: /static
    - path:
        type: Exact
        value: /login
    - path:
        type: Exact
        value: /logout
    - path:
        type: PathPrefix
        value: /api/v1/products
```

:cry: [Issue](https://github.com/istio/istio/issues/51622): unable to push CNI event, error was 500

```bash
# add & restart node
$ cat > /etc/sysctl.d/00-noipv6.conf << EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.eth0.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF
```