:confused: **What is [Relabeling](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config)**?

- A powerful tool to dynamically rewrite the label set of a target **before** it gets scraped.
- Relabels ≈ Rewrite Labels.



:confused: **Why Relabeling?**

- Tag → Aggregate for dev/test/prod env.
- Tag business-related while ignore what we don't care.
- Multi-tenants, collect what teams needed.



:confused: **Meta?**

- Initially
  - A target's `job` label is set to the `job_name` value of the respective scrape configuration. 
  - The `__address__` label is set to the `<host>:<port>` address of the target. 
- After relabeling
  - The `instance` label is set to the value of `__address__` by default if it was not set during relabeling. 
  - The `__scheme__` and `__metrics_path__` labels are set to the scheme and metrics path of the target respectively. 
  - The `__param_<name>` label is set to the value of the first passed URL parameter called `<name>`.
  - The `__scrape_interval__` and `__scrape_timeout__` labels are set to the target's interval and timeout.



:confused: **How?** By [actions](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_action).



:bookmark_tabs: Examples

```yaml
# replace value of label
action: replace
replacement: production
target_label: env

# keep while drop others
action: keep
source_labels: [__address__]
regex: mywebserver01.*

# replace port to 80
action: replace
source_labels: [__address__]
regex: ([^:]+)(?::\d+)?
replacement: "$1:80"
target_保留或丢弃对象 label: __address__

# keep only annotated
action: keep
source_labels:
[__meta_kubernetes_service_annotation_prometheus_scraped]
regex: true
```

