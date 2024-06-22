## [kind](https://kind.sigs.k8s.io/)

```bash
# install
$ go install sigs.k8s.io/kind@v0.23.0
```

```bash
# cluster conf
$ cat > kind-cluster-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: <clusterName>   # here
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
# create cluster
$ kind create cluster --config kind-cluster-config.yaml

# mgmt
$ kind get nodes --name istio-sidecar
$ kind get kubeconfig --name istio-sidecar

# del
$ kind delete cluster --name <name>
```

### [Ingress](https://kind.sigs.k8s.io/docs/user/ingress/)

```bash
# install
$ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml

# apply kind specific patches to forward the hostPorts to the ingress controller
# set taint tolerations and schedule it to the custom labelled node.
$ kubectl patch daemonsets -n projectcontour envoy -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/control-plane","operator":"Equal","effect":"NoSchedule"},{"key":"node-role.kubernetes.io/master","operator":"Equal","effect":"NoSchedule"}]}}}}'
```

```bash
# create
$ kubectl apply -f https://projectcontour.io/examples/httpbin.yaml

# verify
$ kubectl get po,svc,ing -l app=
$ kubectl -n projectcontour port-forward service/envoy 8888:80
# local.projectcontour.io is a pub DNS record resolving to 127.0.0.1
$ curl http://local.projectcontour.io:8888

# cleanup
$ kubectl delete -f https://projectcontour.io/examples/httpbin.yaml
```

### [LB](https://kind.sigs.k8s.io/docs/user/loadbalancer/)

[cloud-proivder-kind](https://github.com/kubernetes-sigs/cloud-provider-kind): a standalone bin in your host and connects to your KIND cluster and provisions new LB containers for your svc.

```bash
# install
$ go install sigs.k8s.io/cloud-provider-kind@latest
$ sudo ln -s $GOPATH/bin/cloud-provider-kind /usr/local/bin/cloud-provider-kind

# allow control node to access workloads on worker using LB svc.
$ kubectl label node <clusterName>-control-plane node.kubernetes.io/exclude-from-external-load-balancers-

# run in another terminal
$ cloud-provider-kind
```

```bash
# create svc & expose in LB
$ kubectl apply -f https://kind.sigs.k8s.io/examples/loadbalancer/usage.yaml

# verify
$ LB_IP=$(kubectl get svc/foo-service -o=jsonpath='{.status.loadBalancer.ingress[0].ip}') && echo $LB_IP

# verify, not applicable for WSL/Mac, still need to use Ingress.
$ for _ in {1..10}; do curl ${LB_IP}:5678; done

# cleanup
$ kubectl delete -f https://kind.sigs.k8s.io/examples/loadbalancer/usage.yaml
```

