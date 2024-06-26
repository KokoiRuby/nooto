### [Ingress](https://istio.io/latest/docs/tasks/traffic-management/ingress/)

Ingress [Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/): entrypoint/lb at the edge of Service mesh. Listener conf into selected Envoy pod. **It only provides L4 access point while L7 is offered by [Virtual Service](https://istio.io/latest/docs/reference/config/networking/virtual-service/).**

- servers: properties of the proxy on a given load balancer port.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "httpbin.example.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```

```bash
# conf in gateway envoy
$ kubectl exec into pod → curl localhost:15000/config_dump

# curl with header
$ curl -H "Host: xxx" gw_svcIP
```

#### [TLS](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/) ([mode](https://istio.io/latest/docs/reference/config/networking/gateway/#ServerTLSSettings-TLSmode))

An HTTPS `Gateway` will perform [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) matching against its configured host(s) before forwarding a request.

- This would fail if u don't have DNS set up & use host header directly `curl 1.2.3.4 -H "Host: app.example.com"`
  - Set DNS or use `--resolve` flag

1. [Generate](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/#generate-client-and-server-certificates-and-keys) keypair.
2. Create secret.

```bash
$ kubectl create -n istio-system secret tls httpbin-credential \
  --key=example_certs1/httpbin.example.com.key \
  --cert=example_certs1/httpbin.example.com.crt
  
# multi-host
$ kubectl create -n istio-system secret tls helloworld-credential \
  --key=example_certs1/helloworld.example.com.key \
  --cert=example_certs1/helloworld.example.com.crt

# mtls
# ++ ca to verify client cert
$ kubectl create -n istio-system secret generic httpbin-credential \
  --from-file=tls.key=example_certs1/httpbin.example.com.key \
  --from-file=tls.crt=example_certs1/httpbin.example.com.crt \
  --from-file=ca.crt=example_certs1/example.com.crt
```

3. Configure Gateway.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: httpbin-credential # ref
    hosts:
    - httpbin.example.com
---
# multi-host
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https-httpbin
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: httpbin-credential # ref
    hosts:
    - httpbin.example.com
  - port:
      number: 443
      name: https-helloworld
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: helloworld-credential # ref
    hosts:
    - helloworld.example.com
---
# mtls
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: MUTUAL                       # mtls
      credentialName: httpbin-credential # ref
    hosts:
    - httpbin.example.com
```

#### [Passthrough](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-sni-passthrough/)

No TLS termination against SNI.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: PASSTHROUGH
    hosts:
    - nginx.example.com
```

#### [Sidecar](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-sidecar-tls-termination/) TLS Termination

On the behalf of Ingress Gateway.

```bash
$ istioctl install --set profile=default --set values.pilot.env.ENABLE_TLS_ON_SIDECAR_INGRESS=true
```

```yaml
# enable global mtls
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
---
# disable authN for externally exposed svc port
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: disable-peer-auth-for-external-mtls-port
  namespace: test
spec:
  selector:
    matchLabels:
      app: httpbin
  mtls:
    mode: STRICT
  portLevelMtls:
    9080:
      mode: DISABLE
```

```bash
# create secret
$ kubectl -n test create secret generic httpbin-mtls-termination-cacert \
	--from-file=ca.crt=./example.com.crt
$ kubectl -n test create secret tls httpbin-mtls-termination \
	--cert ./httpbin.test.svc.cluster.local.crt \
	--key ./httpbin.test.svc.cluster.local.key
```

```yaml
# anble external mTLS for svc envoy.
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: ingress-sidecar
  namespace: test
spec:
  workloadSelector:
    labels:
      app: httpbin
      version: v1
  ingress:
  - port:
      number: 9080
      protocol: HTTPS
      name: external
    defaultEndpoint: 0.0.0.0:80
    tls:
      mode: MUTUAL
      privateKey: "/etc/istio/tls-certs/tls.key"
      serverCertificate: "/etc/istio/tls-certs/tls.crt"
      caCertificates: "/etc/istio/tls-ca-certs/ca.crt"
  - port:
      number: 9081
      protocol: HTTP
      name: internal
    defaultEndpoint: 0.0.0.0:80
```

