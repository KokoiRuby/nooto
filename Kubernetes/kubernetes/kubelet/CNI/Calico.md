:confused: What is [Calico](https://www.tigera.io/project-calico/)?

- A networking and security solution that enables k8s workloads and non-k8s/legacy workloads to communicate seamlessly and securely.



:confused: **vs?**

- L2
  - Bridge         → complexity  ↑
  - VXLAN & Tunnel → cap & decap ↑
- Calico could achieve pure L3
  - Host as Router + BGP + iptables
  - no NAT & Tunnel



:confused: **[Arch](https://docs.tigera.io/calico/latest/reference/architecture/overview)?**

- **Felix**: host agent programs routes & ACLs to achieve connectivity.
- **BIRD**: BGP client to distribute routes from Felix to peer for inter-host routing.
- **confd**: monitors Calico datastore for changes to BGP conf and global defaults. Dynamically gen conf based on updates on DB.
- **CNI plugin**: provides networking for k8s.
- **Typha**: maintains a single datastore connection on behalf of all of its clients like Felix and confd.





![calico-components](https://docs.tigera.io/assets/images/architecture-calico-deae813300e472483f84d6bfb49650ab.svg)



:confused: **How?**

- Same node, L3 BGP routing.
- Inter-node, IP-in-IP tunneling.



:confused: **Deployment?**

- calico-node → initContainer → /opt/cni/bin/install → hostPath

```bash
# calico-kube-controllers
# calico-typha
$ kubectl get deploy -n calico-system

# calico-node
# csi-node-driver
$ kubectl get ds -n calico-system

# hostPath "cni-bin-dir" & "cni-bin-dir"
/opt/cni/bin/
/etc/cni/net.d/
```

```json
# conf: cm "cni-config"
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      # main
        "type": "calico",
      "datastore_type": "kubernetes",
      "mtu": 0,
      "nodename_file_optional": false,
      "log_level": "Info",
      "log_file_path": "/var/log/calico/cni/cni.log",
      # ipam
        "ipam": {
        "type": "calico-ipam",
        "assign_ipv4": "true",
        "assign_ipv6": "false"
      },
      "container_settings": {
        "allow_ip_forwarding": false
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "k8s_api_root": "https://10.96.0.1:443",
        "kubeconfig": "__KUBECONFIG_FILEPATH__"
      }
    },
	# meta
    {
      "type": "bandwidth",
      "capabilities": {
        "bandwidth": true
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```



:confused: **IPAM CRD?** for IP allocation.

```yaml
# kubectl get ippools.crd.projectcalico.org -o yaml
spec:
    allowedUses:
    - Workload
    - Tunnel
    blockSize: 26         # pod CIDR per node.
    cidr: 192.168.0.0/16  # pod CIDR
    ipipMode: Never
    natOutgoing: true
    nodeSelector: all()
    vxlanMode: CrossSubnet
```

```yaml
# kubectl get ipamblocks.crd.projectcalico.org
  spec:
    affinity: host:cncamp
    cidr: 192.168.216.192/26
    attributes:
    - handle_id: xxx        # handle to which pod in which ns on which node
      secondary:
        namespace: 
        node:
        pod:
        timestamp:
    allocated:              # placeholder of ip
    unallocated:
```

```yaml
# kubectl get ipamhandles.crd.projectcalico.org
# handle in detail
spec:
  block:
    192.168.216.192/26: 1
  deleted: false
  handleID: k8s-pod-network.<id>
```



:confused: **How does it look like after?**

```bash
# get container id & name in pod
$ kubectl get po <pod> -o json | jq '.status.containerStatuses[] | .containerID, .name'

# get pid on node
$ crictl inspect -o go-template --template '{{.info.pid}}' <container id>

# check ip & route
$ nsenter -t <pid> -n ip a
$ nsenter -t <pid> -n ip r
$ nsenter -t <pid> -n arping 169.254.1.1   # check which dev respond this IP in route table. ee:...ee

# all cali*@ifN have ee:ee MAC on node
$ ip a

# route on node
$ route -n 
```

```bash
# bird on node
$ ps -ef | grep bird

# conf in calico-node
$ kubectl exec -it calico-node-7hmbt -n calico-system cat /etc/calico/confd/config/bird.cfg

	# each node is vrouter
	# share route among nodes → which podIP should be sent to which node
	router id 192.168.65.180;
	
	protocol direct {
  		debug { states };
  		interface -"cali*", -"kube-ipvs*", "*"; # Exclude cali* and kube-ipvs* but
                                            	# include everything else.  In
                                            	# IPVS-mode, kube-proxy creates a
                                            	# kube-ipvs0 interface. We exclude
                                            	# kube-ipvs0 because this interface
                                            	# gets an address for every in use
                                            	# cluster IP. We use static routes
                                                # for when we legitimately want to
                                                # export cluster IPs.
	}
```

```bash
# chk 
$ iptables-save

# append rule to chain: cali-nat-outgoing
-A cali-nat-outgoing \
-m comment --comment "cali:flqWnvo8yq4ULQLa" \
# match out traffic
-m set --match-set cali40masq-ipam-pools src \
-m set ! --match-set cali40all-ipam-pools dst \
# SNAT
-j MASQUERADE \
# random port
--random-fully
```