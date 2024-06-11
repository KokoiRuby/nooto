:confused: **What is Image Registry?**

- To store/manage/distribute image
- [registry]/[repo]/[imageName]:[tag]
- reg → repo * N → image * M (meta) → Storage (blob)
- OCI Distribution [spec](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#api)



:confused: **Image made up?**

- meta: reg, repo, tag, checksum, layer fd, dockerfile
- blob: entity that form up unioFS



:confused: **vs. ?**

- Public: Open, Convenient, Free-O&M, Low-cost
- Private: Secure, Sensitive, Network-connectivity



:confused: **What is [Harbor](https://goharbor.io/)?**

- An open-source enterprise image reg.
- [Demo Server](https://demo.goharbor.io/account/sign-in?redirect_url=%2Fharbor%2Fprojects)



:confused: [Arch](https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor)?

- Portal → Proxy (Nginx)
  - Core
    - API Server
      - AuthN & AuthZ
      - Middleware: signature & vulnerability check ...
      - API Handlers
    - Config Manager
    - Project Management
    - Quota Manager
    - Chart Controller: proxy to [Chart Museum](https://chartmuseum.com/)
    - Retention Manager: manage tag retension
    - Content Trust: ext. to [Notary](https://github.com/theupdateframework/notary)
    - Replication Controller: HA
    - Scan Manager
    - Notification Manager: webhook
    - OCI Artifact Manager
    - Registry Driver
  - Registry
  - Job Service: queue for asyns task
  - Log Collector
  - GC Controller
  - [Chart Museum](https://chartmuseum.com/): 3pp chart repo server.
  - Docker Registry
  - [Notary](https://github.com/theupdateframework/notary): 3pp trust server to secure content publish & verification.
- Data Access Layer
  - Redis: temp job meta
  - PostgreSQL: Harbor models
  - Blob Storage: persistence for reg & [Chart Museum](https://chartmuseum.com/)



:confused: [Install](https://goharbor.io/docs/2.11.0/install-config/harbor-ha-helm/)? HA → Ingress Controller, PostgresSQL, Redis

### Standard

### Helm