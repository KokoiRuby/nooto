label `istio.io/dataplane-mode=ambient` → individual workload or namespace.

```bash
$ kubectl label ns istio.io/dataplane-mode=ambient
$ kubectl label ns istio.io/dataplane-mode-

$ kubectl label pod istio.io/dataplane-mode=ambient
$ kubectl label pod istio.io/dataplane-mode-
```



**Interoperability** btw pods using ambient & non-ambient endpoints.

- **Outside-the-mesh**
  - Directly to dst. without going through ztunnel.
  - Dst. applies L4 policy whether allow/deny.
- **Inside-the-mesh using sidecar**
  - Sidecar will use HBONE → dst.



**Co-existence**

It is important to ensure that the same pod or ns does not get configured to use both modes at the same time.

- Ssidecar takes precedence when conflicts.

To determine

1. `istio-cni` excludes ns by `cni.values.excludeNamespaces`
2. ns labeled `istio.io/dataplane-mode=ambient` & pod no `istio.io/dataplane-mode=none` & pod no sidecar.istio.io/status annotation.



Label reference

| Name                      | Feature Status | Resource                                     | Description                                                  |
| ------------------------- | -------------- | -------------------------------------------- | ------------------------------------------------------------ |
| `istio.io/dataplane-mode` | Beta           | `Namespace` or `Pod` (latter has precedence) | Add your resource to an ambient mesh. <br />Valid values: `ambient` or `none`. |
| `istio.io/use-waypoint`   | Beta           | `Namespace`, `Service` or `Pod`              | Use a waypoint for traffic to the labeled resource for L7 policy enforcement. <br />Valid values: `{waypoint-name}` or `none`. |
| `istio.io/waypoint-for`   | Alpha          | `Gateway`                                    | Specifies what types of endpoints the waypoint will process traffic for. <br />Valid values: `service`, `workload`, `none` or `all`. <br />This label is optional and the default value is `service`. |

