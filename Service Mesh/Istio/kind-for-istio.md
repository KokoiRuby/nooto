A quickstart to setup a k8s cluster by [kind](https://kind.sigs.k8s.io/).

## Setup

Install kind

```bash
$ go install sigs.k8s.io/kind@v0.23.0
```

Create cluster

```bash
$ cat > kind-cluster-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: istio-sidecar
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
EOF
```

```bash
$ kind create cluster --config kind-cluster-config.yaml
```

Install [Ingress](https://kind.sigs.k8s.io/docs/user/ingress/) (Here I choose [Countour](https://projectcontour.io/getting-started/#test-it-out)). Note: LB does not work in Mac/Windows WSL, see more in this [ref](https://medium.com/groupon-eng/loadbalancer-services-using-kubernetes-in-docker-kind-694b4207575d).

```bash
$ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
$ kubectl patch daemonsets -n projectcontour envoy -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/control-plane","operator":"Equal","effect":"NoSchedule"},{"key":"node-role.kubernetes.io/master","operator":"Equal","effect":"NoSchedule"}]}}}}'
```

Delete cluster

```bash
$ kind delete cluster --name istio-sidecar
```

