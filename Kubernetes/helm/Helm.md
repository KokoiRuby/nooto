[toc]

# [Helm](https://helm.sh/)

Helm → K8s **Package Manager**；Helm Chart groups a collection of resources to start a instance → **Tempaltes** ← Render with **Value**.



Components

- **Helm Client**
  - local chart dev
  - repo/release mgmt
  - interact with Helm Library → send chart to be installed | request upgrade/uninstall of a rel
- **Helm Library** → K8s API Server
  - chart + conf → rel
  - install chart
  - upgrade/uninstall chart



[Install](https://helm.sh/docs/intro/install/)



[Quickstart](https://helm.sh/docs/intro/quickstart/)

```bash
$ helm repo list
# helm repo add azure http://mirror.azure.cn/kubernetes/charts/
$ helm repo add <repoName> <url>
$ helm search repo <chartName>
$ helm search repo <chartName> -l | grep x.y.z
$ helm repo remove <repoName>
```

```bash
$ helm pull azure/mysql --version=1.6.9
$ helm install <releaseName> azure/mysql --version=1.6.9
$ helm ls
$ helm status <relaseName>
$ helm delete <releaseName>
```

```bash
$ tar -zxvf mysql-1.6.9.tgz
$ rm -rf mysql-1.6.9.tgz && helm package mysql/
```
