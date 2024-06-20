:confused: **Security Need since microservices?**

- Man-in-the-middle → **Encryption**.
- Dynamic access control → **mTLS & Fine-grained policies**.
- Who did What at When → **Auditting**.



:confused: **What Istio offers?**

- Security by default: no change to code & infra.
- Defense in depth: multi-layers.
- Zero-trust network



![Security overview](https://istio.io/latest/docs/concepts/security/overview.svg)



:confused: **Arch?**

- CA for certM.

- API: AuthN & AuthZ Policies, [Seucre Naming](https://istio.io/latest/docs/concepts/security/#secure-naming).

  ↓ (Distribute bi-direct gRPC)

- Sidecar as [Policy Enforcement Points](https://csrc.nist.gov/glossary/term/policy_enforcement_point).

- Envoy proxy extensions to manage telemetry & auditing.



![Security Architecture](https://istio.io/latest/docs/concepts/security/arch-sec.svg)



:confused: **Istio Identity?**

- workload-to-workload comm → id info exchange for mTLS.
  - client side: chk against Seucre Naming
  - server side: determine client access control by AuthZ, audit when access what, charge if necessary.
- Support: K8s sa, GCE, On-premises (non-k8s).
  - [spiffe](https://spiffe.io/)://\<domain>/ns/\<namspace>/sa/\<serviceaccount>
  - Istio will check against envoy pod in which ns using which sa. → include uri. & gen cert.



:confused: **CertM?** via Secret DS.

- istio-agent keeps monitoring the expiration of cert on workload, update it periodically.



![Identity Provisioning Workflow](https://istio.io/latest/docs/concepts/security/id-prov.svg)