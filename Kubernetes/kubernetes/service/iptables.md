### [Chains of iptables](https://serenafeng.github.io/2020/03/26/kube-proxy-in-iptables-mode/)

| Chain            | Desc                                                         |
| ---------------- | ------------------------------------------------------------ |
| `KUBE-SERVICES`  | Entrypoint → `KUBE-SVC-*` chain.                             |
| `KUBE-SVC-*`     | LB → `KUBE-SEP-*` chain.                                     |
| `KUBE-SEP-*`     | Service Endpoint, DNAT → podIP:port.                         |
| `KUBE-MARK-MASQ` | Marks packets originate outside cluster and it will be altered in a POSTROUTING rule to use SNAT → nodeIP. |
| `KUBE-MARK-DROP` | Packets with this mark will be dropped in `KUBE-FIREWALL-*` chain. |
| `KUBE-FW-*`      | Match LB VIP → `KUBE-SVC-*` (externalTrafficPolicy: Cluster) & `KUBE-XLB-*` chain (externalTrafficPolicy: Local) |
| `KUBE-NODEPORTS` | Match node port → `KUBE-SVC-*` (externalTrafficPolicy: Cluster) & `KUBE-XLB-*` chain (externalTrafficPolicy: Local) |
| `KUBE-XLB-*`     | externalTrafficPolicy: Local, dropped when no relavent endpoint on node. |



## :bookmark_tabs: Cheatsheet

```bash
$ sudo iptables-save
$ sudo iptables -t nat -S

# in-cluster
$ sudo iptables -t nat -nvL PREROUTING
$ sudo iptables -t nat -nvL KUBE-SERVICES | grep <clusterIP>
$ sudo iptables -t nat -nvL KUBE-SVC-xxx
$ sudo iptables -t nat -nvL KUBE-SEP-xxx
```

