## [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

```bash
# list all api groups
$ kubectl get --raw /apis | jq '.groups[].name'

# to get api group of an api
$ kubectl explain

# list all system:
$ kubectl get clusterroles | grep system:

# dry-run
# https://kubernetes.io/docs/reference/kubectl/generated/kubectl_auth/kubectl_auth_can-i/
$ kubectl auth can-i create pods --as-group=system:nodes
```

```bash
# create role
$ kubectl create role <ROLE_NAME> \
--verb=<VERB> \
--resource=<RESOURCE> \
--namespace=<NAMESPACE>

# create clusterrole
$ kubectl create clusterrole <CLUSTER_ROLE_NAME> \
--verb=<VERB> \
--resource=<RESOURCE>

# create rolebinding
$ kubectl create rolebinding <ROLE_BINDING_NAME> \
--role=<ROLE_NAME> \
--user=<USER_NAME> \
--namespace=<NAMESPACE>

# create clusterrolebinding
$ kubectl create clusterrolebinding <CLUSTER_ROLE_BINDING_NAME> \
--clusterrole=<CLUSTER_ROLE_NAME> \
--user=<USER_NAME> \
--group=<GROUP_NAME> \
--serviceaccount=<NAMESPACE>:<SERVICE_ACCOUNT_NAME>
```

| Verb               | Desc                                            |
| ------------------ | ----------------------------------------------- |
| `get`              | Read-only access to a specific resource.        |
| `list`             | List all instances of a resource.               |
| `watch`            | Watch for changes to a resource.                |
| `create`           | Create a new instance of a resource.            |
| `update`           | Update an existing resource.                    |
| `patch`            | Apply partial updates to a resource.            |
| `delete`           | Delete a specific resource.                     |
| `deletecollection` | Delete a collection of resources.               |
| `impersonate`      | Act as another user, group, or service account. |
| `bind`             | Bind roles to subjects in RBAC.                 |
| `escalate`         | Grant permission to escalate privileges.        |