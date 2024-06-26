#### [Traffic redirection](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/)

Data plane functionality that intercepts traffic sent to and from ambient-enabled workloads, routing it through the ztunnel node proxies

- `istio-cni` pod (node agent) responds to CNI events such as pod creation/deletion & write rules into pod network ns.
- `istio-cni` node agent then informs the ztunnel proxy, over a Unix domain socket, that it should establish local proxy listening ports inside the pod’s network namespace. (15008, 15006, 15001)
- The node-local ztunnel internally spins up a new proxy instance and listen port set, dedicated to the newly-added pod.
- Once the in-pod redirect rules are in place and the ztunnel has established the listen ports, the pod is added in the mesh and traffic begins flowing through the node-local ztunnel.
- **Traffic to and from pods in the mesh will be fully encrypted with mTLS by default.**



![pod added to the ambient mesh flow](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/pod-added-to-ambient.svg)



![HBONE traffic flows between pods in the ambient mesh](https://istio.io/latest/docs/ambient/architecture/traffic-redirection/traffic-flows-between-pods-in-ambient.svg)