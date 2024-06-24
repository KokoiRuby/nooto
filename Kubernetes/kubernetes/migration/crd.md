:confused: **To extend?** [vs](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#comparing-ease-of-use)

- **Aggregation** (API Server)
  - Generate types obj
  - Generate clientset, lister, informer by client-go
  - Impl regustry backed = define how obj is stored in etcd
  - Reg scheme to apiserver.
  - Create obj api declaration & reg to api handler.
- **CRD-based Operator**
  - Reg a resource with a customized schema.
  - CRD → CR ← Controller watches & takes action.
  - ++ RBAC
  - [sample-controller](https://github.com/kubernetes/sample-controller)
  - [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)