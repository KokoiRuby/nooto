:confused: **What is [gRPC proxy](https://etcd.io/docs/v3.5/op-guide/grpc_proxy/)?**

- A stateless L7 reverse proxy for etcd.
- Aggregates watch & lease API. (Fron N to 1)
- Caches key range requests.
- Protects against abusive client.



:confused: **How does it work?**

- Randomly pick one server endpoint, swtich if detects failure.



:confused: **Name Resolv?**

- It allows clients to synchronize their endpoints against a set of proxy endpoints for high availability.
- `--resolver-prefix`: Reg endpoints for discovery vis sync

-  `--resolver-ttl`



:confused: **Namespacing?**

- `--namespace`: all client req going into the proxy are **translated to a user-defined prefix** on keys.



:confused: **Metrics?**

- `--metrics-addr` on `/metrics` & `/health`