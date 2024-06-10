:confused: **[Sidecar Injection](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/)?**

- Auto

  ```bash
  $ kubectl label namespace <namespace> istio-injection=enabled --overwrite
  $ kubectl label pod <pod> sidecar.istio.io/inject=true
  ```

- Manual

  ```bash
  $ istioctl kube-inject -f yaml/to/apply.yaml | kubectl apply -f -
  ```



:confused: **What happened then?** 

- sample/sleep (as client) → sample/httpbin (as server)

  ```bash
  $ kubectl apply -f samples/sleep/sleep.yaml
  $ kubectl apply -f samples/httpbin/httpbin.yaml
  ```

- ++ initContainer "istio-init"

  ```bash
  # rule injection
  istio-iptables
  -p "15001"
  -z "15006"
  -u "1337"
  -m REDIRECT
  -i '*'
  -x
  -b
  '*'
  -d 15090,15021,15020
  ```

  ```bash
  # host where sleep is
  $ crictl pods | grep sleep
  $ crictl inspectp <podid> | grep pid
  $ nsenter -t <pid> -n iptables-save
  ```

  ```bash
  # outgoing
  # from app container: OUTPUT → ISTIO_OUTPUT → ISTIO_REDIRECT where Envoy sidecar is listening on
  -A OUTPUT -p tcp -j ISTIO_OUTPUT
  -A ISTIO_OUTPUT -s 127.0.0.6/32 -o lo -j RETURN
  -A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -p tcp -m tcp ! --dport 15008 -m owner --uid-owner 1337 -j ISTIO_IN_REDIRECT
  -A ISTIO_OUTPUT -o lo -m owner ! --uid-owner 1337 -j RETURN
  # from envoy sidecar: OUTPUT → ISTIO_OUTPUT → RETURN (to OUTPUT)
  -A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
  -A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -p tcp -m tcp ! --dport 15008 -m owner --gid-owner 1337 -j ISTIO_IN_REDIRECT
  -A ISTIO_OUTPUT -o lo -m owner ! --gid-owner 1337 -j RETURN
  -A ISTIO_OUTPUT -m owner --gid-owner 1337 -j RETURN
  -A ISTIO_OUTPUT -d 127.0.0.1/32 -j RETURN
  -A ISTIO_OUTPUT -j ISTIO_REDIRECT
  -A ISTIO_REDIRECT -p tcp -j REDIRECT --to-ports 15001
  ```

  ```bash
  # incoming
  # PREROUTING → ISTIO_INBOUND → ISTIO_IN_REDIRECT where Envoy sidecar is listening on
  -A PREROUTING -p tcp -j ISTIO_INBOUND
  -A ISTIO_INBOUND -p tcp -m tcp --dport 15008 -j RETURN
  -A ISTIO_INBOUND -p tcp -m tcp --dport 15090 -j RETURN
  -A ISTIO_INBOUND -p tcp -m tcp --dport 15021 -j RETURN
  -A ISTIO_INBOUND -p tcp -m tcp --dport 15020 -j RETURN
  -A ISTIO_INBOUND -p tcp -j ISTIO_IN_REDIRECT
  -A ISTIO_IN_REDIRECT -p tcp -j REDIRECT --to-ports 15006
  ```

- ++ sidecar container "istiod"

  - **listener → route → cluster → endpoint**

  ```bash
  # listener
  
  # envoy 15001 → envoy 80
  
  # "useOriginalDst": true → keep dst IP to forward
  # "trafficDirection": "OUTBOUND"
  $ istioctl pc listener -n default sleep-5577c64d7c-qsvb2 --port 15001 -o json
  # # "routeConfigName": "80"
  $ istioctl pc listener -n default sleep-5577c64d7c-qsvb2 --port 80 -o json
  ```

  ```bash
  # route
  # "name": "httpbin.org:80"
  # "route" → "cluster": "outbound|8000|httpbin.default.svc.cluster.local"
  $ istioctl pc route -n default sleep-5577c64d7c-qsvb2 --name 80 -o json 
  ```

  ```bash
  # cluster
  # httpbin.default.svc.cluster.local
  $ istioctl pc cluster -n default sleep-5577c64d7c-qsvb2
  ```

  ```bash
  # endpoint
  $ istioctl pc endpoint -n default sleep-5577c64d7c-qsvb2 | grep httpbin.default.svc.cluster.local
  ```

  





