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



:confused: **Limitation of K8s svc & ingress?**

- svc     → L4 (IP:port) only, limited LB strategy, does not support keep-alived conn
- ingress → L7 but HTTP/HTTPS only, does not support gRPC, limitation LB strategy like header-based.



:confused: **What happened now?**

- Calls btw svc → Calls btw Sidecar Proxies.



<img src="overview.assets/image-20240610141634062.png" alt="image-20240610141634062" style="zoom: 67%;" />



:confused: **What is Service Mesh?**

- A network topo made by these Sidecar Proxies.



<img src="overview.assets/image-20240610141725134.png" alt="image-20240610141725134" style="zoom: 67%;" />



:confused: **Pros vs. Cons?**

- :smile: Business Logic & Platform Features decoupled; Developers only needs to focus Business Logic.

- :cry: Complexity ↑ | Extra hop in kernel → un-friendly to High-concurrency & Delay-sensitive.



:confused: **What is Istio ⛵?**

- An impl of Service Mesh, an platform for providing a uniform way to:
  - integrate microservices.
  - manage traffic flow across.
  - enforce policy.
  - aggregate telemetry data.



:confused: **Istio Arch?**

- **Envoy** as Data Plane
- **Istiod** as Control Plane
  - **Pilot** - Responsible for <u>configuring</u> the proxies at runtime.
  - **Citadel** - Responsible for <u>certificate</u> issuance and rotation.

  - **Galley** - Responsible for validating, ingesting, aggregating, transforming and distributing config within Istio.



![The overall architecture of an Istio-based application.](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)



:confused: **Istio Features?**



<img src="overview.assets/Istio.png" alt="img" style="zoom: 50%;" />

