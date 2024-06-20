:confused: **Arch?**



![Authentication Architecture](https://istio.io/latest/docs/concepts/security/authn.svg)



:confused: **Types?**

- **Peer**: svc-to-svc in mtls with CertM → gen/dist/rotate.
  - client side checks against secure naming presented in server cert.
    - [DestinationRule.spec.trafficPolicy.tls](https://istio.io/latest/docs/reference/config/networking/destination-rule/#TrafficPolicy)
  - server side determines which AuthN to accept.
    - [PeerAuthentication](https://istio.io/latest/docs/reference/config/security/peer_authentication/)
  - min. TLSv1_2.
  - ++ Headers
    - `X-Fowarded-Client-Cert` → URI=spiffe://cluster.local/ns/\<ns>/sa/\<sa>
- **Request**: end-user cred (not managed by Istio) attached to request by JWT.
  - As long as Envoy can resolve JWT, traffic can be passed-through.
  - [RequestAuthentication](https://istio.io/latest/docs/reference/config/security/request_authentication/)
    - selector → gateway
    - issuer & jwks (key set) → resolv jwt token



:confused: **Permisssive Mode?** Default

- Allow svc to accept both plaintext and mTLS traffic at the same time.
- Flexibility for onboarding.



:confused: **Secure Naming?**

- server identities encoded in cert, while service names are retrieved by svc discovery or DNS.
- It maps server identities → service names.
  - A → svc B = A is authorized to run svc B.



