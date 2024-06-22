:confused: **What is [IPVS](https://demonlee.tech/archives/2306001)?**

- IP Virtual Server, which impl L4 LB inside kernel.
- Clients → IPVS (on host) → Real Servers.



:confused: **Hooks in netfilter?**

- Input
  - `ip_vs_remote_request` → `ip_vs_in`: handle packet into IPVS (from remote).
  - `ip_vs_reply` → `ip_vs_out`: handle packet out IPVS (to clients), SNAT.
- Foward
  - `ip_vs_forward_icmp` → handle ICMP packet into IPVS.
  - `ip_vs_reply` → `ip_vs_out`: handle segment out IPVS (to clients), SNAT.
- Output
  - `ip_vs_local_request` → `ip_vs_in`: handle packet into IPVS (from local).
  - `ip_vs_local_reply` → `ip_vs_out`: handle packet out IPVS (to clients), MASQ = Dynamic SNAT.



![img](https://gitlab.com/jwkljh/img2023/-/raw/main/2306/ipvs-on-netfilter.png)



:confused: **kube-proxy mode change?**

```bash
# mode → ipvs
$ kubectl edit configmap kube-proxy -n kube-system

# restart kube-proxy pod
# backup flusth iptables
$ iptables-save
$ iptables --flush
```

### :bookmark_tabs: Cheatsheet

```bash
# install
$ sudo apt-get install ipvsadm

# all svc IPs bind to NIC, ping-able
$ ip a show kube-ipvs0
```

```bash
# rules
$ iptables -t nat -nvL PREROUTING
↓
# rules in KUBE-SERVICES → KUBE-CLUSTER-IP ipset
$ iptables -t nat -nvL KUBE-SERVICES 
↓
$ ipset list KUBE-CLUSTER-IP
↓
# locate backend real servers by clusterIP
$ ipvsadm -l
$ ipvsadm -ln
```

```bash
# add v svc
$ ipvsadm -A -t 192.168.20.128:80 -s wrr

# add backend
$ ipvsadm -a -t 192.168.20.128:80 -r 192.168.10.131:80 -m -w 1
$ ipvsadm -a -t 192.168.20.128:80 -r 192.168.10.132:80 -m -w 2
```

