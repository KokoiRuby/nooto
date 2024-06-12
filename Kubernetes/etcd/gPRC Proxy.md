:confused: **What is gRPC proxy?**

- A stateless L7 reverse proxy for etcd.
- Aggregates watch & lease API. (Fron N to 1)
- Caches key range requests.
- Protects against abusive client.



:confused: **How does it work?**

- Randomly pick one server endpoint, swtich if detects failure.