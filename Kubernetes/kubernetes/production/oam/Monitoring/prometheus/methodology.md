:confused: **Google Golden Signals?**



![Golden Signals PPT Slide 1](https://cdn.sketchbubble.com/pub/media/catalog/product/optimized1/1/f/1f18166a67d98841ab95f299913d3955f8d56ec27e98bb65ddb9538c1a1108ab/golden-signals-mc-slide1.png)



:confused: **The "RED" Method?** Primarily aimed at **request-driven** systems → microservice friendly.

- **R**ate (the number of requests per second)
- **E**rrors (the number of those requests that are failing)
- **D**uration (the amount of time those requests take)



:confused: **USE Method?** Primary aimed at **hardware**, network disks

- **U**tilization (% time that the resource was busy)
- **S**aturation (amount of work resource has to do, often queue length)
- **E**rrors (count of error events)



:confused: **Monitoring Levels？**

- **Business**: order, transaction, # of online users.
  - Prevision: [Medium]
  - Real-time Responsiveness: [High]
  - Take it sercirouly, abnormal → Considered as Fault :exclamation:
- **Application**: "Gloden Signal" & "RED"
  - [APM](https://www.dynatrace.com/news/blog/what-is-apm-2/) - Application Performance Management.
  - Instrumentation - invasive vs. Non-invasive.
- **Component**: DB, Middle-part, Platform
  - Unified data access spec like Prometheus.
- **Resource**: 
  - Device
    - Server: CPUI, Mem, Disk, IO, HW → IPMI
    - Netdev: SW, Firewall → SNMP
  - Network
    - Connectivity: ICMP
    - Quality: Drop rate, Latency
    - Traffic: NIC



:confused: How to scrape?

- Exec
  - /proc → need secondary computation.
  - commandline with pipe.
- Remote
  - Black: ICMP | TCP | HTTP
    - Stats by sending multiple packets: [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter), [Categraf](https://github.com/flashcatcloud/categraf),  [Datadog-Agent](https://www.datadoghq.com/)
    - Scheme endpoint: Elasticsearch | RabbitMQ | Kubelet
  - White
    - The monitored exposes HTTP → protocol format like Prometheus.
    - Run cmd like MySQL, Redis (meanwhile fetch business stats).



:confused: **Instrumentation?**

- Invasive: SDK/lib
- Non-invasive: Java-agent

| Mode    | :smile:      | :cry:                     |
| ------- | ------------ | ------------------------- |
| Sidecar | Flexible.    | Twin cost.                |
| Central | Cost-saving. | Changed rules affect all. |



:confused: **Log extraction?**

- Scrapee: rule run on host directly, matched numeric as metric to report.
  - [mtail](https://github.com/google/mtail) | [grok_exporter](https://github.com/fstab/grok_exporter)
- Central: App → Kafka → Elasticsearch