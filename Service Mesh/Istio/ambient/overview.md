Ambient ≈ “Sidecar-less”, per-node L4 + optinoal per-namesapce L7.

`Waypoint proxioes` above

- Envoy Deployment outside application pods.
- L7 traffic enforcement.

`ztunnel` as base

- Per-node daemon.
- L3/L4 → mTLS, AuthN, AuthZ, Telemetry.
- [HBONE](https://istio.io/latest/docs/ambient/architecture/hbone/) tunneling protocol.



![img](https://pic4.zhimg.com/80/v2-ecb1273afa276cd60f4b8702b6cdc0c7_1440w.webp)



![img](https://pic4.zhimg.com/80/v2-557038dde4fc69b60f7f91fdb99a4e7b_1440w.webp)

#### [Architecture](https://istio.io/latest/docs/ambient/architecture/)

**Control Plane:** ztunnel xDS client & CA client → Istiod for conf & cert (of dst app).

- **Pull (on-demand)** > Push (from Istiod → sidecars) → Istiod Pressure ↓
- Less invasive.



<img src="https://istio.io/latest/docs/ambient/architecture/control-plane/ztunnel-architecture.png" alt="Ztunnel architecture" style="zoom: 50%;" />



**Data Plane：**

Workload Category

1. **Out of Mesh**: a standard pod without any mesh features enabled.
2. **In Mesh**: a pod that is included in the ambient data plane, and has traffic intercepted at the Layer 4 level by ztunnel.
3. **In Mesh, Waypoint enabled**: a pod that is *in mesh* *and* has a waypoint proxy deployed. In this mode, L7 policies can be enforced for pod traffic.



In Mesh Routing

- Outbound
  - redirected to node-local ztunnel
  - if In Mesh, req → HBONE
  - if In Mesh, Waypoint enabled, req → HBONE → L7 polices.
- Inbound
  - redirected to node-local ztunnel for AuthN.
  - When the destination is waypoint enabled, if the source is in ambient mesh, the source’s ztunnel ensures the request **will** go through the waypoint where policy is enforced. Otherwise, it is sent directly to.



![Basic ztunnel L4-only datapath](https://istio.io/latest/docs/ambient/architecture/data-plane/ztunnel-datapath-1.png)

