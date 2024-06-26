### [Egress](https://istio.io/latest/docs/tasks/traffic-management/egress/)

#### [Access Ext. Service](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/)

[ServiceEntry](https://istio.io/latest/docs/reference/config/networking/service-entry/): enables adding additional entries into Istio’s internal service registry, so that auto-discovered services in the mesh can access/route to these manually specified (external) services.

- host: IP/FQDN to

```bash
# ALLOW_ANY (default) vs REGISTRY_ONLY

# set
$ istioctl install <flags-you-used-to-install-Istio> --set meshConfig.outboundTrafficPolicy.mode=...
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  location: MESH_EXTERNAL
---
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: google
spec:
  hosts:
  - www.google.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
  location: MESH_EXTERNAL
```

```bash
# verify
$ curl -sS http://httpbin.org/headers
$ curl -sSI https://www.google.com | grep  "HTTP/"
```

Similar to inter-cluster requests, routing rules can also be configured for external services that are accessed using `ServiceEntry` configurations. = ++ Virtual Service

```yaml
kind: VirtualService
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  http:
  - timeout: 3s # ++ timeout
    route:
    - destination:
        host: httpbin.org
      weight: 100
```

Direct access = bypass Istio for a specific IP range = No Envoy intercepting. Get range from cluster provider.

- `global.proxy.includeIPRanges`
- `global.proxy.excludeIPRanges`

#### [Egress TLS Origination](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-tls-origination/)

Istio can open HTTPS connections to the external service. ++ Destination Rule

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: edition-cnn-com
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 80
    name: http-port
    protocol: HTTP
  - number: 443
    name: https-port
    protocol: HTTPS
  resolution: DNS
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: edition-cnn-com
spec:
  host: edition.cnn.com
  trafficPolicy:
    portLevelSettings:
    - port:
        number: 80
      tls:
        mode: SIMPLE # initiates HTTPS when accessing edition.cnn.com
```

if [mtls](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-tls-origination/#mutual-tls-origination-for-egress-traffic)

```bash
# server keypair secret
$ kubectl create -n mesh-external secret tls nginx-server-certs \
	--key my-nginx.mesh-external.svc.cluster.local.key \
	--cert my-nginx.mesh-external.svc.cluster.local.crt
$ kubectl create -n mesh-external secret generic nginx-ca-certs \
	--from-file=example.com.crt


# client keypair with ca
$ kubectl create secret generic client-credential \
	--from-file=tls.key=client.example.com.key \
	--from-file=tls.crt=client.example.com.crt \
	--from-file=ca.crt=example.com.crt
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: originate-mtls-for-nginx
spec:
  hosts:
  - my-nginx.mesh-external.svc.cluster.local
  ports:
  - number: 80
    name: http-port
    protocol: HTTP
    targetPort: 443
  - number: 443
    name: https-port
    protocol: HTTPS
  resolution: DNS
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: originate-mtls-for-nginx
spec:
  workloadSelector:  # select workload which is client
    matchLabels:
      app: sleep
  host: my-nginx.mesh-external.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    portLevelSettings:
    - port:
        number: 80
      tls:
        mode: MUTUAL
        credentialName: client-credential 
        sni: my-nginx.mesh-external.svc.cluster.local # this is optional
```

```bash
# verify
$ istioctl proxy-config secret deploy/sleep | grep client-credential
```

#### [Gateway](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/)

Egress gateway defines exit points from the mesh.

**Jump *2: sidecar → egress gateway → ext. svc.**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 80
    name: http-port
    protocol: HTTP
  resolution: DNS
---
# if tls
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - edition.cnn.com
---
# if tls
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 443
      name: tls
      protocol: TLS
    hosts:
    - edition.cnn.com
    tls:
      mode: PASSTHROUGH
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-cnn
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: cnn
```

**mesh** is a special reserved gateway name in Istio that represents all the sidecar proxies within the mesh.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-cnn-through-egress-gateway
spec:
  hosts:
  - edition.cnn.com
  gateways:
  - istio-egressgateway
  - mesh
  http:
  # if coming from sidecar, will match "mesh" gateway & we direct to egress gateway
  - match:
    - gateways:
      - mesh
      # port: 443 # if tls
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: cnn
        port:
          # port: 443 # if tls
          number: 80
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      # port: 443 # if tls
      port: 80
    route:
    - destination:
        host: edition.cnn.com
        port:
          # port: 443 # if tls
          number: 80
      weight: 100
```

Use Network Policy to limit egress from ns to contonrol plane.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-istio-system-and-kube-dns
spec:
  podSelector: {}  # all pods
  policyTypes:
  - Egress
  # only allow UDP 53 → kube-system ns.
  egress:          
  - to:
    - namespaceSelector:
        matchLabels:
          kube-system: "true"
    ports:
    - protocol: UDP
      port: 53
  # allows pods to send traffic to any service in the istio namespace.
  - to:             
    - namespaceSelector:
        matchLabels:
          istio: system
```

#### [Gateway TLS Origination](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway-tls-origination/)

Sidecar → HTTP → Gateway → HTTPS → Ext. svc.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 80
    name: http
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 80
      name: https-port-for-tls-origination
      protocol: HTTPS
    hosts:
    - edition.cnn.com
    tls:
      mode: ISTIO_MUTUAL
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-cnn
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: cnn
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
      portLevelSettings:
      - port:
          number: 80
        tls:
          mode: ISTIO_MUTUAL
          sni: edition.cnn.com
EOF
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-cnn-through-egress-gateway
spec:
  hosts:
  - edition.cnn.com
  gateways:
  - istio-egressgateway
  - mesh
  http:
  - match:
    - gateways:
      - mesh
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: cnn
        port:
          number: 80
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      port: 80
    route:
    - destination:
        host: edition.cnn.com
        port:
          number: 443
      weight: 100
EOF
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: originate-tls-for-edition-cnn-com
spec:
  host: edition.cnn.com
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    portLevelSettings:
    - port:
        number: 443
      tls:
        mode: SIMPLE # initiates HTTPS for connections to edition.cnn.com
```

if [mtls](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway-tls-origination/#perform-mutual-tls-origination-with-an-egress-gateway)

```bash
# server keypair secret
$ kubectl create -n mesh-external secret tls nginx-server-certs \
	--key my-nginx.mesh-external.svc.cluster.local.key \
	--cert my-nginx.mesh-external.svc.cluster.local.crt
$ kubectl create -n mesh-external secret generic nginx-ca-certs \
	--from-file=example.com.crt


# client keypair with ca
$ kubectl create secret generic client-credential \
	--from-file=tls.key=client.example.com.key \
	--from-file=tls.crt=client.example.com.crt \
	--from-file=ca.crt=example.com.crt
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - my-nginx.mesh-external.svc.cluster.local
    tls:
      mode: ISTIO_MUTUAL
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-nginx
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: nginx
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
      portLevelSettings:
      - port:
          number: 443
        tls:
          mode: ISTIO_MUTUAL
          sni: my-nginx.mesh-external.svc.cluster.local
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-nginx-through-egress-gateway
spec:
  hosts:
  - my-nginx.mesh-external.svc.cluster.local
  gateways:
  - istio-egressgateway
  - mesh
  http:
  - match:
    - gateways:
      - mesh
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: nginx
        port:
          number: 443
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      port: 443
    route:
    - destination:
        host: my-nginx.mesh-external.svc.cluster.local
        port:
          number: 443
      weight: 100
```

```yaml
kubectl apply -n istio-system -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: originate-mtls-for-nginx
spec:
  host: my-nginx.mesh-external.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    portLevelSettings:
    - port:
        number: 443
      tls:
        mode: MUTUAL
        credentialName: client-credential # this must match the secret created earlier to hold client certs
        sni: my-nginx.mesh-external.svc.cluster.local
        # subjectAltNames: # can be enabled if the certificate was generated with SAN as specified in previous section
        # - my-nginx.mesh-external.svc.cluster.local
EOF
```

```bash
$ istioctl -n istio-system proxy-config secret deploy/istio-egressgateway | grep client-credential
```

#### [Wildcard Hosts](https://istio.io/latest/docs/tasks/traffic-management/egress/wildcard-egress-hosts/)

Access `*.wikipedia.org`

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: wikipedia
spec:
  hosts:
  - "*.wikipedia.org"
  ports:
  - number: 443
    name: https
    protocol: HTTPS
```

via gateway

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: www-wikipedia
spec:
  hosts:
  - www.wikipedia.org  # configured with the host of the single server for the set of domains.
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - "*.wikipedia.org"
    tls:
      mode: PASSTHROUGH
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-wikipedia
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
    - name: wikipedia
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-wikipedia-through-egress-gateway
spec:
  hosts:
  - "*.wikipedia.org"
  gateways:
  - mesh
  - istio-egressgateway
  tls:
  - match:
    - gateways:
      - mesh
      port: 443
      sniHosts:
      - "*.wikipedia.org"
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: wikipedia
        port:
          number: 443
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      port: 443
      sniHosts:
      - "*.wikipedia.org"
    route:
    - destination:
        host: www.wikipedia.org
        port:
          number: 443
      weight: 100
EOF
```