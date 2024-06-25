### [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/)

- **Filter**

  - Instance Query "斬" きり

    - `--query.lookback-delta` (5min), shorter like 1min to avoid alarm triggered too frequently (for). 

    ```sql
    -- filter by Label
    node_load1{zone="sh"}
    node_load1{host=~"host0.*"}
    
    -- __name__ hidden key to filter.
    {__name__=~"node_load.*", zone="sh"}
    ```

  - Range Query　"AOE": based freq, if scrape/10s, them 6 samples in 1min.

    ```sql
    -- ++[1m]
    {__name__=~"node_load.*", zone="sh"}[1m]
    ```

- **Secondary Computation**

  - Arithmetic

    ```sql
    mem_available{app="xxx"} / mem_total{app="xxx"} * 100
    irate(net_sent_bytes_total{zone="beijing"}[1m]) * 8
    ```

  - Compare

    ```sql
    mem_available{app="clickhouse"} / mem_total{app="clickhouse"} * 100 < 20
    ```

  - Logical

    ```sql
    disk_used_percent{app="xxx"} > 70 and disk_total{app="xxx"}/1024/1024/1024 > 200
    ```

  - Vector Matching: join-like

    ```sql
    -- Check slave_sql_running is MySQL instance is a slave.
    -- match by label "instace"
    mysql_slave_status_slave_sql_running == 0
    and ON (instance)
    mysql_slave_status_master_server_id > 0
    
    method_code:http_errors:rate5m{method="get", code="500"}  24
    method_code:http_errors:rate5m{method="get", code="404"}  30
    method_code:http_errors:rate5m{method="put", code="501"}  3 method_code:http_errors:rate5m{method="post", code="500"} 24
    method_code:http_errors:rate5m{method="post", code="404"} 21
    method:http_requests:rate5m{method="get"} 600
    method:http_requests:rate5m{method="del"} 34
    method:http_requests:rate5m{method="post"} 120
    
    method_code:http_errors:rate5m{code="500"}
    / ignoring(code)
    method:http_requests:rate5m
    
    {method="get"}  0.04  # 24 / 600
    {method="post"} 0.05  # 6  / 120
    ```

    ```sql
    method_code:http_errors:rate5m
    / ignoring(code) group_left
    method:http_requests
    
    {method="get", code="500"}  0.04  # 24 / 600
    {method="get", code="404"}  0.05  # 30 / 600
    {method="post", code="500"} 0.05  # 6  / 120
    {method="post", code="404"} 0.175 # 21 / 120
    ```

- **Aggregation** = Longitudinal Fitting = N points → 1 point

  - Sort

    ```sql
    avg(mem_available_percent{app="xxx"})
    topk(2, mem_available_percent{app="xxx"})
    bottomk(2, mem_available_percent{app="xxx"})
    ```

  - Grouping

    ```sql
    avg(mem_available_percent{app="xxx"} by (app)
    ```

- **Horizontal Fitting**

  - accept range vector

    ```sql
    max_over_time(target_up[2m])
    ```

    

:confused: **vs?**

- [increase](https://prometheus.io/docs/prometheus/latest/querying/functions/#increase): accept a range vector → **increment** within time duration.

  ```sql
  -- sample/10s
  net_bytes_recv{interface="eth0"}[1m]
  965304237246 @ 1661570850
  965307953982 @ 1661570860
  965311949925 @ 1661570870
  965315732812 @ 1661570880
  965319998347 @ 1661570890
  965323899880 @ 1661570900
  
  increase(net_bytes_recv{interface="eth0"}[1m])
  
  ```

  

- rate: accept a range vector → **delta** within time duration.

  - [rate](https://prometheus.io/docs/prometheus/latest/querying/functions/#rate) = increase / size(range-vector)
  - Smooth spikes, for data changed frequently, use [irate](https://prometheus.io/docs/prometheus/latest/querying/functions/#irate).

  ```sql
  rate(net_bytes_recv{interface="eth0"}[1m])
  -- =
  increase(net_bytes_recv{interface="eth0"}[1m])/60.0
  ```

  

