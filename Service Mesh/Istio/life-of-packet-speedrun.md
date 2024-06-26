### [Part 1](https://dramasamy.medium.com/life-of-a-packet-in-istio-part-1-8221971d77de)

- **Service Mesh**: a dedicated **infra** that you can add to your applications. It allows you to **transparently** add capabilities like observability, traffic management, and security, **without** adding them to your own code.

- **Istio**: Impl of Service Mesh → Traffic Management, Observablity, Policy Enforcement, Security.
  
  - Control Plane (istiod) + Data Plane (envoy)
  
    
  
- **Sidecar Injection**

  - istio-init → iptables rules injection
    - runAsUser:0
    - --all but ++ NET_ADMIN & NET_RAW cap → privilege to iptables.
  - istio-proxy → user:group = 1337:1337
  



- **iptables**

  - outgoing

    - OUTPUT → ISTIO_OUTPUT → ISTIO_REDIRECT → 15001 envoy listener

      ↓

    - OUTPUT → ISTIO_OUTPUT → RETURN (to OUTPUT)

  - incoming

    - PRERROUTING → ISTIO_INBOUND → ISTIO_IN_REDIRECT → 15006 envoy listener
    - OUTPUT → ISTIO_OUTPUT → RETURN (to PRERROUTING/INPUT)

### [Part 2](https://dramasamy.medium.com/life-of-a-packet-in-istio-envoy-proxy-part-2-27950e820b04)

- Envoy is designed to work as a sidecar proxy, running alongside each workload in a service mesh.

  1. `Listener` accepts incoming conn from downstream on specific ports & protocols.

  2. `Listener Filter Chain` manipulates conn meta. (SNI & pre-TLS details)

  3. `Network Filter Chain` is used to match and offload TLS.

  4. `HTTP Filter Chain` is used for HTTP codec.

     - `Router Fileter` selects a upstream cluster based on req headers.

  5. `Cluster` performs LB & selects an endpoint.

  6. `Endpoint Connection` codec/encrypt and write them to a TCP socket.

  7. Req is proxied to upstream & passed to HTTP Filter Chain in reverse order.

     

<img src="https://miro.medium.com/v2/resize:fit:875/1*s3MxPjCOF7EM25Fc6ccB_Q.png" alt="img" style="zoom: 80%;" />



### [Part 3](https://dramasamy.medium.com/life-of-a-packet-in-istio-control-plane-part-3-899e565da4d2)