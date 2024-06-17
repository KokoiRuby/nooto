:construction_worker: **Practice**

- AuthN: disable anonymous access.
  - ext. Microsoft Active Directory
  - Webhook
  - Keystone
- AuthZ: avoid conflict among, such as normal user deletes k8s core svc, user A delete user B's resources.
  - See more in Overview.md
- Isolation
  - Visiability: user can only see what they create by themselves.
  - Resource: some dedicated not sharable.
  - App
- Quota: Who can use how much?