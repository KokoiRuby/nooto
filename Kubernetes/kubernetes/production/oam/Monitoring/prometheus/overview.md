:confused: **Why Monitoring?**

- To sense whether system is functioning well, business is serving well.
- Prediction by tread.
- Waterlevel ref. to scaling.
- Ref. to optimication.



:confused: **Monitoring Category?**



![image-20240624140038748](./overview.assets/image-20240624140038748.png)



:confused: **vs?**

|                  | Push                                        | Pull                              |
| ---------------- | ------------------------------------------- | --------------------------------- |
| Impl             | InfluxDB                                    | Prometheus                        |
| Initiator        | Monitored                                   | Monitor                           |
| Network          | Dst. IP fixed. Need to pass firewall.       | Need to connect to all monitored. |
| Concurrency      | Pressure to                                 | Round-robin                       |
| Fault Detection  | Monitored not awared, keep sending → crash. | Getting slow if overload.         |
| Target Discovery | No                                          | Yes                               |



:confused: **Observability?** Metrics/Logs out → Monitroing & Tracing.

- Metrics: Prometheus, InfluxDB, Zabbix.
- Logging: Loki, ELK.
- Tracing: Jaeger, Zipkin, Skywalking.



:confused: **Metric Type?**

- **Global ID String**: (Zabbix)

  - :smile: Simple.
  - :cry: No extra dimention, need appending to name, messy.

  ```json
  // 每 30s 采集主机已使用内存
  {
      "name": "host.10.2.3.4.mem_used_percent",
      "points": [
          {
              "clock": 1662449136,
              "value": 45.4
          },
          ...
      ]
  }
  ```

- **LabelSet** (Prometheus, OpenTSDB)

  - name + timestamp + k/v label

  ```bash
  mysql.bytes_received 1287333217 327810227706 schema=foo host=db1
  ```

- **Influx** (InfluxDB)

  - measurement(,tag_set), field_set, timestamp(ns)

  ```bash
  mysql,schema=foo,host=db1 bytes_received=327810227706,bytes_sent=6604859181710 
  ```



:confused: **Genearl Arch?**

- **Collector**: coupled with the monitored vs. centralized probe → the monitored.
- **TSDB**: OpenTSDB, InfluxDB, TDEngine, M3DB, VictoriaMetrics, TimescaleDB.
- **Alert Engine**: generates & sends alert by rules. Data-triggered vs. Polling
- **Display**: Grafana → Ad-hoc + Dashboard.



:confused: **What is Prometheus?**

- An open-source systems <u>monitoring & alerting</u> toolkit.



:confused: **Prometheus [Arch](https://prometheus.io/docs/introduction/overview/#architecture)?** 

- Prometheus server
  - Retrival: Target Discovery & Scaper to.
  - TSDB: Persistent.
- [Push Gateway](https://github.com/prometheus/pushgateway): for Short-lived jobs.
- Exporter: for Visulization.
- [AlertManager](https://github.com/prometheus/alertmanager): for Alerting.



![image-20240624140256612](./overview.assets/image-20240624140256612.png)



:confused: [**Service Discovery](https://prometheus.io/docs/prometheus/latest/storage/#operational-aspects)?** Static Stock → Dynmiac in Cloud Native

- conf-based.

  :smile: simple yaml, easy to automate, .

  :cry: Isolated island → solved by federation.

- K8s SD.

- Public-cloud API.

- Consul.



:confused: **Data Model?**

- Metrics (name + k/v *N) → Sample(value + timestamp) * N
- `__name__` is a reserved label that represents the metric name or id of a time series.



![image-20240624141524455](./overview.assets/image-20240624141524455.png)



:confused: **Metric [Type](https://prometheus.io/docs/concepts/metric_types/)?**

- Counter ++ only, 0 when reset.
- Gauge ++/--
- Histogram (Data Distribution)
  - counter in Bucket
  - quantile calculated in Server, would overload/OOM if too many scrapee.
  - (upper - lower) * (curr / total) + lower
- Summary (Data Distribution)
  - quantile calculated in Client side then → Server



:confused: **In K8s?**

- Scrape → kubelet (cAdvisor) for host/pod/container metrics

```yaml
# annotate & port
prometheus.io/port: http-metrics
prometheus.io/scrape: "true"
# ...
spec:
  ports:
  - containerPort: 3100
    name: http-metrics
    protocol: TCPs
```

```go
// reg handler for endpoint
http.Handle("/metrics", promhttp.Handler())
http.ListenAndServe(sever.MetricsBindAddress, nil)

// reg metrics
func RegisterMetrics() {
    registerMetricOnce.Do(func() {
    	prometheus.MustRegister(APIServerRequests)
    	prometheus.MustRegister(WorkQueueSize)
    })
}

// export
metrics.AddAPIServerRequest(controllerName,
                            constants.CoreAPIGroup, 
                            constants.SecretResource,
                            constants.Get,cn.Namespace)
```



:confused: **Recording [Rule](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#recording-rules)?**

- It allsows to precompute frequently needed or computationally expensive expressions and save their result as a new set of time series. 
- Querying the precomputed result will then often be much **faster** than executing the original expression every time it is needed.