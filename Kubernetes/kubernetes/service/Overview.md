:confused: **To publish a svc?**

- Type?
- CertM & L7 LB
- How to perform gRPC LB?
- DNS
- Downstream & Upstream



:confused: **K8s primitive challenges?**

- svc
  - ClusterIP internal only
  - kube-proxy iptables/ipvs → scale/perf/prod/drift = (acutal ≠ desired)
  - floating pod IP → LB pool change
  - ELB integration
  - not support gRPC
  - not support customize DNS & advanced routing.
- ingress
  - spec maturity...



:confused: **Cross-region?**

- How many workloads?
- How to setup failure domain?
- How many regions? Avail Zones? cluster?
- How to perform precise traffic management?
- How to update region by region?
- How to perform rollback?