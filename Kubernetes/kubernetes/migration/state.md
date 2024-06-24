:confused: **Stateless?**

- Deployment
  - version control: `deployment.kubernetes.io/revision` in annotation (++ when updated), `.spec.revisionHistoryLimit`
  - update: `.spec.strategy`



:confused: **Stateful?**

- ++ `.spec.serviceName` & `.spec.volumeClaimTemplates`