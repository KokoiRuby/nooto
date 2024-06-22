### [Listeners](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners_toc)

- TCP
  - static resuorces → listner * N, each configured with [filter_chain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-filter-chains) (composed of L3/4 [filters](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listener_filters#arch-overview-network-filters)) * N + [default](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-default-filter-chain), each based on [filter_chain_match](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchainmatch).
  - When a new conn → listener, the appropriate filter_chain is selected & **conn-local** filter stack is instantiated & begins to proc, such as [rate limit](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting#arch-overview-global-rate-limit), [TLS client AuthN](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/security/ssl#arch-overview-ssl-auth-filter), [HTTP conn mgmt](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/http/http_connection_management#arch-overview-http-conn-man)...
- UDP
  - [filters](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/udp_filters/udp_filters#config-udp-listener-filters), instantiated once per worker

### [Listener filters](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listener_filters)

- To manipulate conn meta, adding further sys integration functions easier.
- Filters are chained in FilterChain (most [match](https://www.envoyproxy.io/docs/envoy/latest/xds/type/matcher/v3/matcher.proto#envoy-v3-api-msg-xds-type-matcher-v3-matcher) criteria), that can be updated independetly.
- Network (L3/4) filters
  - R: invoked when Envoy receives data from downstream.
  - W: invoked when Envoy is about to send data to downstream.
  - R/W: invoked when both.
  - TCP/UDP proxy & DNS fitler.

## HTTP

### Connection Management

- Built-in network level filter: HTTP connection manager.

### Filters

### Routing

### Upgrade

### HTTP/3