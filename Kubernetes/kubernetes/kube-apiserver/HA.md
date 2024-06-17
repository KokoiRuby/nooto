:confused: **How?**

- API Server itself is a stateless REST Server. Easy to scale in/out.
- Typical → Redundancy + LB (cert needs extra VIP/SAN).



:construction_worker: **Practice**

- Preserve enough CPU/Mem.

- Rate Limit Flags & APF.

- API Server ← (HTTP/2 based gRPC) → etcd。

  - stream quota → concurrency

- `--watch-cache-sizes` while client better set resourceVersion to avoid cache miss.

  ```bash
  GET /api/v1/namespaces/test/pods?watch=1&resourceVersion=<...>
  ```

- large obj under high concurrency → client ListWatch to listen in keep-alived instead of full fetch.
- Merge multi-informer to decrease conn to API Server.
- Alwasy to API Server by LB VIP.