:confused: **Why Ingress?**

- L4
  - ELB each VIP per svc, cost too much.
  - EDNS each record per svc, hard to maintain.
- L7
  - multi-svc shared one ELB VIP & EDNS record.
  - TLS termination, centralized CertM.
  - Complexity ↑, hop++



:confused: **What is [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)?**

- A proxy layer, forward traffic to svc by hostname & path.
- Ingress defines rules of forwarding.
- Ingress Controller listening & apply conf.



:confused: **Challenge?**

- tls: cipher, dhkey, TLSVersion
- Header-based rule.
- Header/URI rewrite.



:smile: [Contour](https://projectcontour.io/) & [Istio](https://istio.io/)