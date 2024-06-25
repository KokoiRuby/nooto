:confused: **[Install](https://prometheus.io/docs/prometheus/latest/installation/)?**

- [Setup](https://github.com/KokoiRuby/docker/tree/main/prometheus)

| Flag                     | Desc                                                      |
| ------------------------ | --------------------------------------------------------- |
| `--config.file`          | /path/to/prometheus.yml                                   |
| `--storage.tsdb.path`    | Base path for metrics storage. Use with server mode only. |
| `--web.enable-lifecycle` | Enable shutdown and reload via HTTP request.              |
| `--enable-feature`       | Comma separated feature names to enable.                  |
| `--query.lookback-delta` | The maximum lookback duration for retrieving metrics      |
| `--web.enable-admin-api` | Enable API endpoints for admin control actions.           |