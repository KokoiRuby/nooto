:confused: **What is kubectl?**

- A cmd tool to interact with K8s cluster.
- It will transform client request into REST API to API Server



:confused: [Install](https://kubernetes.io/docs/tasks/tools/)?



:confused: **[~/.kube/config](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)**

```bash
# view
kubectl config view

# specify 
kubectl ... --kubeconfig=.kube/config

# add cluster into kubeconfig
kubectl config set-cluster <cluster> \
--server=<url> \
--certificate-authority=<path to>.crt
--embed-certs=true

# add user into kubeconfig with cred
kubectl config set-credentials <user> \
--client-certificate=<crt>.crt \
--client-key=<key>.key \
--embed-certs=true

# add context - user <-> cluster
kubectl config set-context <user>@<cluster name> \
--user=<user> \
--cluster=<cluster name>

# switch context
kubect config use-context <user>@<cluster name>
```



:bookmark_tabs: [Cheatsheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

```bash
$ kubectl get po -o[yaml|json|wide] -w
$ kubectl describe
$ kubectl exec
$ kubectl logs
```

