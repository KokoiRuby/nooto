### [Part 1](https://dramasamy.medium.com/life-of-a-packet-in-kubernetes-part-1-f9bc0909e051)

- **Linux Namespaces**

  - Global system **resources isolation** btw independent proc.

  - mnt, uts, ipc, pid, **net**, user, cgroup

    

- **Container networking**

  ```bash
  # create net ns
  $ ip netns add client
  $ ip netns add server
  $ ip netns list
  ```

  ```bash
  # add link
  $ ip link add veth-client type veth peer name veth-server
  $ ip link list | grep veth
  
  # put to each net ns
  $ ip link set veth-client netns client
  $ ip link set veth-server netns server
  $ ip link list | grep veth
  
  # into ns
  $ ip netns exec client ip link
  $ ip netns exec server ip link
  ```

  ```bash
  # ipam
  $ ip netns exec client ip address add 10.0.0.11/24 dev veth-client
  $ ip netns exec client ip link set veth-client up
  
  $ ip netns exec server ip address add 10.0.0.12/24 dev veth-server
  $ ip netns exec server ip link set veth-server up
  
  # verify
  $ ip netns exec client ping 10.0.0.12 -c 3
  $ ip netns exec server ping 10.0.0.11 -c 3
  ```

  ```bash
  $ BR=bridge
  
  # add link
  $ ip link add client1-veth type veth peer name client1-veth-br
  $ ip link add server1-veth type veth peer name server1-veth-br
  $ ip link add $BR type bridge
  
  # create ns
  $ ip netns add client1
  $ ip netns add server1
  
  # put one end to each net, the other bridge
  $ ip link set client1-veth netns client1
  $ ip link set server1-veth netns server1
  $ ip link set client1-veth-br master $BR
  $ ip link set server1-veth-br master $BR
  
  # up link
  $ ip link set $BR up
  $ ip link set client1-veth-br up
  $ ip link set server1-veth-br up
  $ ip netns exec client1 ip link set client1-veth up
  $ ip netns exec server1 ip link set server1-veth up
  
  # ipam
  $ ip netns exec client1 ip addr add 172.30.0.11/24 dev client1-veth
  $ ip netns exec server1 ip addr add 172.30.0.12/24 dev server1-veth
  $ ip addr add 172.30.0.1/24 dev $BR
  
  # verify
  $ ip netns exec client1 ping 172.30.0.12 -c 3
  $ ip netns exec server1 ping 172.30.0.11 -c 3
  $ ip netns exec client1 ping 172.30.0.1 -c 3
  $ ip netns exec server1 ping 172.30.0.1 -c 3
  ```

  ```bash
  $ HOST_IP=172.17.0.31
  
  # add default route to bridge
  $ ip netns exec client1 ip route add default via 172.30.0.1
  $ ip netns exec server1 ip route add default via 172.30.0.1
  
  # verify
  $ ip netns exec client1 ping $HOST_IP -c 3
  $ ip netns exec server1 ping $HOST_IP -c 3
  ```

- **CNI**

  - CNCF → **spec and lib** for writing plugins to conf net interfaces in containers.

  - CR reads conf from CNI dir & loads plugin **exec** to setup container network via CNI (ADD/DEL).

    - `--cni-conf-dir`: `/etc/cni/net.d`

    - `--cni-bin-dir`: `/opt/cni/bin`

      

- **Pod net namespace**

  - pause container provides nets while others join by nsenter.

### [Part 2](https://dramasamy.medium.com/life-of-a-packet-in-kubernetes-part-2-a07f5bf0ff14)

- **CNI Requirement**
  1. Create veth-pair and move the same inside container
  2. Identify the right POD CIDR
  3. Create a CNI conf file
  4. Assign and manage IP address
  5. Add default routes inside the container
  6. Advertise the routes to all the peer nodes (Not applicable for VxLan)
  7. Add routes in the HOST server
  8. Enforce Network Policy



- **Calico comp**
  - **Bird**: a per-node BGP daemon that **exchanges route** info with BGP daemons running on other nodes.
    - Messy if large scale → Route Reflector reduce BGP sessions instead of peering to every node.
  - **Felix**: programs routes (routing table) & ACLs (iptables/ipvs) to achieve connectivity.
  - **confd**: listens & reads conf from DB (etcd) & assembles them for BIRD to use.



- **How the packet gets routed to the peer node?**

  - pod → arping → default gateway 169.254.1.1 in routing table → which dev on host shall respond.

    ```bash
    # switch
    $ cat /proc/sys/net/ipv4/conf/cali*/proxy_arp
    ```

    

- **[Overlay networking](https://docs.tigera.io/calico/latest/networking/configuring/vxlan-ipip)**

  - IP-in-IP (default)

    - inner header with pod src/dst IPs

      ↓

      (tunl0)

      ↓

    - outer header with host src/dst IPs

  - Direct

    - no (de)cap overhead, highly performant → latency-sensitive :smile:

  - VxLAN

    - inner header with pod src/dst IPs

      ↓

    - UDP

      ↓

    - outer header with host src/dst IPs

      

- [Install](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises) & [calicoctl](https://docs.tigera.io/calico/latest/reference/calicoctl/)

  ```bash
  # ++
  # /etc/cni/net.d
  10-calico.conflist
  
  # /opt/cni/bin
  calico
  calico-ipam
  ```

  ```bash
  $ calicoctl node status
  $ calicoctl get workloadendpoints
  ```

### [Part 3](https://dramasamy.medium.com/life-of-a-packet-in-kubernetes-part-3-dd881476da0f)

- **Pod-to-Pod**

  - without NAT by CNI plugin, kube-proxy is not involved.

- **Pod-to-External**

  - SNAT: podIP → nodeIP

- **Pod-to-Service** ← kube-proxy

  - coreDNS lookup: `svcName.ns.svc.domainName`
  - svcIP:port → podIP:targetPort

- **External-to-pod** (`externalTrafficPolicy`: Cluster | Local)

  - **NodePort**

    - `--service-node-port-range`
    - anyClusterNode:nodePort → podIP:port

    

- **kube-proxy**

  - iptables

    - DNAT **ctstate** to remember the dst. addr it changed to & change it back when returning the packet.

      - NEW
      - ESTABLISHED
      - RELATED (affiliation, like FTP)
      - INVALID

    - Chains/Hooks

      - Incoming to local: `PREROUTING` (net.ipv4.ip_forward) → `INPUT`
      - Incoming to another: `PREROUTING` (net.ipv4.ip_forward) → `FORWARD` → `POSTROUTING`
      - Outgoing to: OUTPUT → `POSTROUTING`

    - Table

      - filter: allow/deny → firewall
      - nat: nat (new conn)
      - managle
      - raw

    - ClusterIP: `KUBE-SERVICES` → `KUBE-SVC-*` → `KUBE-SEP-*`

    - NodePort: `KUBE-SERVICES` → `KUBE-NODEPORTS` → `KUBE-SVC-*` → `KUBE-SEP-*`

      

- **Headless**: clusterIP is Nonde

  - not handled by kube-proxy
  - stateful/ExternalName
  - coreDNS lookup: `podName.svcName.ns.svc.domainName`

  

- **NetworkPolicy**: regulate in/out traffic to/from pod.

  - iptables "filter" table

### [Part 4](https://dramasamy.medium.com/life-of-a-packet-in-kubernetes-part-4-4dbc5256050a)

- Ingress: L7 LB → path/host based.
- LB: VIP allocation.
