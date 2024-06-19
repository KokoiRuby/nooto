:confused: What is [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)?

- An application-centric construct which allow you to specify how a [pod](https://kubernetes.io/docs/concepts/workloads/pods/) is allowed to communicate with various network **"entities"** which identified as
  - Other pods
  - Namespaces
  - IP Blocks
- L3/4-based.
- **Need network plugin (like Calico) to impl (defined policies)** ≈ Ingress.
- Union (Ingress & Exgress)



:confused: **spec?**

- `podSelector`: label-based → polciy enforce on which pods.
- `policyTypes`: ingress as default if not specified, must set Egress if had.
- `ingress`: whitelist, from(ipBlock/namespaceSlector/podSelector) & ports
- `egress`: whitelist, to(ipBlock/namespaceSlector/podSelector) & ports



:confused: **Default?**

```yaml
# deny all in
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-in
spec:
  podSelector: {}
  policyTypes:
    - Ingress
---
# deny all out
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-out
spec:
  podSelector: {}
  policyTypes:
    - Egress
---
# allow all in
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-in
spec:
  podSelector: {}
  ingress:
    - {}
  policyTypes:
    - Ingress
---
# allow all out
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-out
spec:
  podSelector: {}
  egress:
    - {}
  policyTypes:
    - Egress
```



:confused: **Calico [Network Policy](https://docs.tigera.io/calico/latest/reference/resources/networkpolicy)？**

- Advanced.

- apiVersion: projectcalico.org/v3

- ++[GlobalNetworkPolicy](https://docs.tigera.io/calico/latest/reference/resources/globalnetworkpolicy) effective for all ns, can also limit pod-to-host.

  ```yaml
  apiVersion: projectcalico.org/v3
  kind: GlobalNetworkPolicy
  metadata:
    name: allow-ping-in-cluster
  spec:
    selector: all()
    types:
      - Ingress
    ingress:
      - action: Allow
        protocol: ICMP
        source:
          selector: all()
        icmp:
          type: 8 # Ping request
      - action: Allow
        protocol: ICMPv6
        source:
          selector: all()
        icmp:
          type: 128 # Ping request
  ---
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-http
    namespace: calico-demo
  spec:
    podSelector: {}
    ingress:
      - from:
          - namespaceSelector:
              matchLabels:
                kubernetes.io/metadata.name: default
        ports:
          - protocol: TCP
            port: 80
  ```

- Controller in calico-node

  ```bash
  # Apply & check diff
  # filter table → firewall
  # nat table    → svc
  $ sudo iptables-save
  $ sudo iptables -t nat -L -nv
  $ sudo iptables -t filter -L -nv
  
  # define chain
  :cali-INPUT - [0:0]
  # + chain into INPUT = all incoming will jump to
  -A INPUT -m comment --comment "cali:Cz_u1IQiXIMmKD4c" -j cali-INPUT
  # -i cali+ regex match NIC name → ip a
  -A cali-INPUT -i cali+ -m comment --comment "cali:H03xYXARh4e8pwd4" -g cali-wl-to-host
  -A cali-wl-to-host -m comment --comment "cali:Ee9Sbo10IpVujdIY" -j cali-from-wl-dispatch
  
  -A cali-from-wl-dispatch -i cali4+ -m comment --comment "cali:2b7OEVWxujqPNMC9" -g cali-from-wl-dispatch-4
  -A cali-from-wl-dispatch-4 -i cali44ea6404663 -m comment --comment "cali:MDogRbZQzU84Q8QF" -g cali-fw-cali44ea6404663
  
  # if conn is in RELATED,ESTABLISHED accept
  -A cali-fw-cali44ea6404663 -m comment --comment "cali:-Qmxi8IRueXsd7tR" -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
  # mark
  -A cali-fw-cali44ea6404663 -m comment --comment "cali:hAztzaqicHQ0BTXu" -j MARK --set-xmark 0x0/0x10000
  # return if had mark
  -A cali-fw-cali44ea6404663 -m comment --comment "cali:ar6L67t9axgz62IV" -m comment --comment "Return if profile accepted" -m mark --mark 0x10000/0x10000 -j RETURN
  ```



:confused: **Debugging?**

- iptables
- tcpdump → blocked at which NIC
- dmsg