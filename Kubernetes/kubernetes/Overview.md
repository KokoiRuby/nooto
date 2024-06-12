:confused: **What is Cloud?**

- A distributed collection of servers that deliver services over the Internet.
- Workloads over Platform.



:confused: **Cloud Service Model?**

- IaaS → vResource
- PasS → Runtime, DevTool, DB, Identity AuthN
- SaaS → App



![img](https://www.redhat.com/rhdc/managed-files/styles/wysiwyg_full_width/private/iaas-paas-saas-diagram5.1-1638x1046.png?itok=Ul0Em30u)



:confused: **Distributed System Arch?**

| Type           | Characteristics |                                                              |
| -------------- | --------------- | ------------------------------------------------------------ |
| Centralized    | Control/Workers | Borg → [Kubernetes](https://kubernetes.io/)， Apache [[Mesos](https://mesos.apache.org/)] |
| De-centralized | Peers/Members   |                                                              |



:confused: **What is Kubernetes?**

- An open source container orchestration engine for automating deployment, scaling, and management of containerized applications.



:confused: **Features?**

- Automation
- SD & LB
- Elasticity
- Rolling Update
- Extensibility with plugins & hooks
- Idempotent
- Object-oritend



:confused: **vs.?**

- Imperative: "How to do"
- **Declarative: "Do what" ← K8s**



:confused: **[Arch](https://kubernetes.io/docs/concepts/architecture/)?**

- Master
  - **API Server**
    - REST API → AAA
    - Cluster entrypoint
    - THE only allower to etcd (local-cache)
  - **etcd**
    - Distributed k/v db to store cluster obj status
    - Key watch & TTL
    - Distributed lock & Leader election
  - **Controller Manager**
    - "Brain"
    - Controller of "*Controller"
    - Actual → Desired
    - Eventual Consistency
  - **Scheduler**
    - Predict → Priority → Bind
- Worker
  - **Kubelet**
    - Reg Node to cluster
    - Listening to req from API Server
    - Pod LCM
    - Res & Health report
  - **Kube-proxy**
    - Listening svc/ep & update node iptables
    - Impl svc → pod LB



<img src="https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg" alt="Components of Kubernetes" style="zoom:50%;" />
