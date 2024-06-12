:confused: **Ecosys?**

<img src="./HLD.assets/ecosys.drawio.png" alt="ecosys.drawio" style="zoom:50%;" />

:confused: **Design?**

<img src="./HLD.assets/image-20240612130029855.png" alt="image-20240612130029855" style="zoom:50%;" />

:confused: **Layered?**

- **Ecosystem**
  - Ext.: Logging, Monitoring, Conf Management, CICD, Workflow ...
  - Int.: CRI/CNI/CSI, Reg, Cloud Provider. Identity Provider
- **Interface**: kubectl, SDK, Cluster Federation
- **Governance**: Scale, Automation, Strategy (RBAC, Quota, NetworkPolicy...)
- **Application**: Deploy, Routing (SD, DNS)
- **Nuclues**: API & Hook



<img src="./HLD.assets/image-20240612130223206.png" alt="image-20240612130223206" style="zoom: 50%;" />



:confused: **API?** vs: API is repr, while API Object is a (persistent) record of intent/desire.

- Declarative.
- High cohesion (group functions) & Loose coupled (less dep).
- Business-oriented.
- Low-level reused by High-level.
- Complexity < O(N), otherwise :no_entry: horizontal scale.
- API Object status does not depend on network.



![image-20240612132006488](./HLD.assets/image-20240612132006488.png)



:confused: **Principle?**

- Only API Server can access etcd, other components talks to API Server to get cluster status.
- Single-point Failure shall not affect cluster status.
- Event listening (non-blocking) > Polling (blocking)