:confused: **Best Practice?**

- Build: split conf from code, don't add any sensitives.

- Dep: keep necessity only, make sure no security risk.

- File: don't use `CMD rm ...` to fix problematic file.

- Vulnerability Scan

- Admission Control: [ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admis sion-controllers/#imagepolicywebhook) Allow/Deny againt image scan svc.

  



:confused: **vs?**

- 