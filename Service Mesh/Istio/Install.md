## [Sidecar](https://istio.io/latest/docs/setup/install/)

### istioctl

```bash
# download istioctl
$ curl -L https://istio.io/downloadIstio | sh -

# add istioctl to PATH
$ cd istio-*/bin
$ export PATH=$PWD/bin:$PATH
```

```bash
# install
$ istioctl install --set profile=demo -y
```

```bash
# list avail profile to install
$ istioctl profile list

# diff profile
$ istioctl profile p1 p2

# dump profile
$ istioctl profile dump demo

# dump config
$ istioctl profile dump --config-path components.pilot demo
$ istioctl profile dump --config-path components.galley demo
$ istioctl profile dump --config-path components.citadel demo

# gen manifest
$ istioctl manifest generate > istio-manifest.yaml
```

```bash
# uninstall
$ istioctl uninstall --purge 
```

### Helm

```bash
# add repo
$ helm repo add istio https://istio-release.storage.googleapis.com/charts
$ helm repo update
```

```bash
# create ns
$ kubectl create ns istio-system
$ kubectl create ns istio-ingress

# install
$ helm install istio-base istio/base -n istio-system --set defaultRevision=default
$ helm install istiod istio/istiod -n istio-system --wait
$ helm install istio-ingress istio/gateway -n istio-ingress --wait

# chk
$ helm ls -n istio-system
```

```bash
# uninstall
$ helm delete istio-ingressgateway -n istio-ingress
$ helm delete istiod -n istio-system
$ helm delete istio-base -n istio-system

$ kubectl delete ns istio-ingress istio-ingress
```

## [Ambient](https://istio.io/latest/docs/ambient/install/)

### Helm

## [Bookinfo Demo](https://istio.io/latest/docs/examples/bookinfo/)

```bash
# install
$ kubectl label namespace default istio-injection=enabled
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml

# verify
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" \-c ratings \
	-- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

```bash
$ export INGRESS_NAME=istio-ingressgateway
$ export INGRESS_NS=istio-system

$ export INGRESS_HOST=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
$ export TCP_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')

$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

# verify
$ curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```

```bash
# add entries to C:\Windows\System32\drivers\etc
127.0.0.1 bookinfo.istio

$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-bookinfo
  namespace: istio-system
spec:
  virtualhost:
    fqdn: bookinfo.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF
```

Verify: http://bookinfo.istio/productpage

```bash
# cleanup
$ samples/bookinfo/platform/kube/cleanup.sh

$ cat << EOF | kubectl delete -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-bookinfo
  namespace: istio-system
spec:
  virtualhost:
    fqdn: bookinfo.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF
```

### [Telemery Addons](https://istio.io/latest/docs/tasks/observability/gateways/#option-2-insecure-access-http)

```bash
# WSL/Mac
$ export INGRESS_DOMAIN="telemetry.istio"

$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-telemetry-kiali
  namespace: istio-system
spec:
  virtualhost:
    fqdn: kiali.telemetry.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF

$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-telemetry-prometheus
  namespace: istio-system
spec:
  virtualhost:
    fqdn: prometheus.telemetry.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF

$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-telemetry-grafana
  namespace: istio-system
spec:
  virtualhost:
    fqdn: grafana.telemetry.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF

$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-telemetry-tracing
  namespace: istio-system
spec:
  virtualhost:
    fqdn: tracing.telemetry.istio
  routes:
    - conditions:
        - prefix: /
      services:
        - name: istio-ingressgateway
          port: 80
EOF
```

Verify:

- http://grafana.telemetry.istio
- http://prometheus.telemetry.istio
- http://kiali.telemetry.istio
- http://tracing.telemetry.istio

```bash
# It doesn't work...
# https://projectcontour.io/docs/1.21/config/request-rewriting/#header-rewriting
$ cat << EOF | kubectl apply -f - 
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: istio-telemetry
  namespace: istio-system
spec:
  virtualhost:
    fqdn: telemetry.istio
  routes:
    - conditions:
        - prefix: /kiali
      services:
        - name: istio-ingressgateway
          port: 80
      pathRewritePolicy:
        replacePrefix:
        - prefix: /kiali
          replacement: /
      requestHeadersPolicy:
        set:
        - name: Host
          value: kiali.telemetry.istio
EOF
```

- http://telemetry.istio/kiali



