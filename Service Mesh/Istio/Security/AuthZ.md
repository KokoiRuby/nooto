:confused: **Arch?**

- Each Envoy is a AuthZ engine, determines whether to ALLOW or DENY.



![Authorization Architecture](https://istio.io/latest/docs/concepts/security/authz.svg)



:confused: **Overview?**

- mesh-, namespace-, and workload-wide.
- Flexible semantics: CUSTOM, DENY, ALLOW actions.
- High compatibility: gRPC, HTTP, HTTPS, HTTP/2, and TCP as well.
- [AuthorizkationPolicy](https://istio.io/latest/docs/reference/config/security/authorization-policy/)
  - selector → Policy Target
  - action
  - rules
    - from (sources)
    - to (fined-grained, HTTP verb & path)
    - when



:confused: **Preceduence?**

![Authorization Policy Precedence](https://istio.io/latest/docs/concepts/security/authz-eval.svg)