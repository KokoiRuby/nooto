:confused: **Service Discovery?**

- Introduced by mircoservice arch, need a machanism to comm in svc topo.
- svc provider (cluster + LB) → reg ← svc consumer.



:confused: **LB Feature?** As reverse Proxy

- Even traffic → Throughput ↑
- Failover → HA
- Elasticity → ++/--
- Secure, Black/White List



:confused: **Type?**

- Centralized
  - consumer (by DNS) → LB (NAT & HC) ← provider * N
  - :smile: simple | mainstream | easy to access control
  - :cry: single-point, bottleneck, hop cost
- In-proc
  - (consumer + LB) → reg (HA) ← provider * N keepAlive
  - (consumer + LB) → provider * N
  - :smile: no hop cost
  - :cry: client lib couples with business logic
- Standalone = "Soft"
  - (consumer | LB) → reg (HA) ← provider * N keepAlive
  - LB split from consumer, as standalone proc on host.
  - :smile: no single-point, LB proc down only affects one host consumer.



:confused: **Essense of LB?**

- **Modify headers of transfer unit of network protocols.**
  - TCP/UDP Termination (record orig client IP) 
  - NAT
  - Mac (Direct Routing)
  - Tunnel (IP-in-IP)