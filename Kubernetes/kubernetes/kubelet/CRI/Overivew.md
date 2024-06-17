:confused: **What is CRI?**

- A group of gRPC svc → Image (pull/list/del) & Runtime (container LCM + exec/attach/port-forward).

- **Kubelet (grpc client) → CRI → Container Runtime (grpc server).**

- Orchestrator → Engine → (gRPC) High-level Runtim

- e → (shim) → (OCI) Low-level Runtime.

  - [Docker](https://www.docker.com/), [podman](https://podman.io/)
  - [containerd](https://containerd.io/), [cri-o](https://cri-o.io/)
  - [runc](https://github.com/opencontainers/runc), [gVisor](https://gvisor.dev/), [katacontainers](https://katacontainers.io/)

  

![img](https://miro.medium.com/v2/resize:fit:1050/1*_x0ujgxNUyzBIco_J6O-mw.png)



​	![image-20240617211402681](Overivew.assets/image-20240617211402681.png)



:confused: **vs?** Containerd > cri-o.



![image-20240617211645243](Overivew.assets/image-20240617211645243.png)



:confused: **How to check which CR is in-use?**

- kubelet flag: `--container-runtime-endpoint`