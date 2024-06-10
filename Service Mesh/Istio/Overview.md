:confused: **Arch Evol?**

- Monolithic → single App Server (Modules * N) + Redundancy ← Admin Server Deployment & Configuration.

  ↓

- MicroService → svc * N 

  :heavy_plus_sign:  API Gateway (API Aggregation).

  :heavy_plus_sign:  Service Registry (Service Discovery).

  :heavy_plus_sign:  Circuit Breaker (Fast-Failure).

  :heavy_plus_sign:  LB (HA).

  :heavy_plus_sign:  AuthN & AuthZ (Identity & Access Control).



:confused: **Feature Boundary?**

- Decouple :heavy_plus_sign: from business logic to platfrom → Sidecar Proxy (as delegator).



:confused: **What happened now?**

- Calls btw svc → Calls btw Sidecar Proxies.



<img src="Overview.assets/image-20240610141634062.png" alt="image-20240610141634062" style="zoom: 67%;" />



:confused: **What is Service Mesh?**

- A network topo made by these Sidecar Proxies.



<img src="Overview.assets/image-20240610141725134.png" alt="image-20240610141725134" style="zoom: 67%;" />



:confused: **Pros vs. Cons?**





