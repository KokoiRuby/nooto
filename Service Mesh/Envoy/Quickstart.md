:confused: **[Install](https://www.envoyproxy.io/docs/envoy/latest/start/install#)?**

```bash
$ docker run -itd \
--name envoy \
-p 9901:9901 \
-p 10000:10000 \
envoyproxy/envoy:v1.30-latest

# clean up
$ docker rm -f envoy
```



:confused: **[Run](https://www.envoyproxy.io/docs/envoy/latest/start/quick-start/run-envoy)?**

```bash
# chk
curl http://localhost:9901/
curl http://localhost:10000/

# exec into
$ /usr/local/bin/envoy --version
$ /usr/local/bin/envoy --help
$ /usr/local/bin/envoy --config-path /path/to/envoy-custom.yaml
$ /usr/local/bin/envoy --config-yaml /path/to/envoy-override.yaml
$ /usr/local/bin/envoy --mode validate -c /etc/envoy/envoy.yaml
$ /usr/local/bin/envoy --log-path /path/to/custom.log
$ /usr/local/bin/envoy --log-level [trace|debug|info|warn|error|critical|off]
$ /usr/local/bin/envoy --component-log-level comp:level[,comp:level]
```



:confused: [Admin](https://www.envoyproxy.io/docs/envoy/latest/start/quick-start/admin)?

- admin port

- static_resources.filter_chains.filters.stat_prefix

- [config_dump](http://localhost:9901/config_dump)

  ```bash
  $ curl -s http://localhost:9901/config_dump | jq -r '.configs[] | .["@type"]'
  $ curl -s http://localhost:9901/config_dump?resource=dynamic_listeners | jq '.configs[0].active_state.listener.address'
  ```

- [stats](http://localhost:9901/stats)

  ```bash
  $ curl -s http://localhost:9901/stats | cut -d. -f1 | sort | uniq
  $ curl -s http://localhost:9901/stats?filter='^http\.ingress_http'
  ```