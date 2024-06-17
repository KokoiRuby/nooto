### [kubectx](https://github.com/ahmetb/kubectx?tab=readme-ov-file#installation)

```bash
$ sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
$ sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
$ sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

### [minikube](https://minikube.sigs.k8s.io/docs/)

```bash
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

```bash
$ minikube start

# in-case blocked
$ minikube start \
	--image-mirror-country=cn 
	--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

# chk
$ kubectl get po -A

$ minikube dashboard
```

```bash
# svc
$ kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
$ minikube service hello-minikube
# http://localhost:7080/
$ kubectl port-forward service/hello-minikube 7080:8080
```

```bash
# lb
$ kubectl create deployment balanced --image=kicbase/echo-server:1.0
$ kubectl expose deployment balanced --type=LoadBalancer --port=8080

# in another terminal
$ sudo minikube tunnel

# get ext. IP
# http://<extIP>:8080
$ kubectl get services balanced
```

```bash
# ingress
$ minikube addons enable ingress
$ kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml

# in another terminal
$ sudo minikube tunnel

# get ip
$ kubectl get ingress
$ curl http://192.168.0.2/foo
$ curl http://192.168.0.2/bar
```

```bash
# Mgmt
$ minikube [pause|unpause]
$ minikube stop
$ minikube config set memory 9001
$ minikube addons list
# https://kubernetes.io/releases/
$ minikube start -p aged --kubernetes-version=v1.16.1
$ minikube delete --all
```

### [kind](https://kind.sigs.k8s.io/)

```bash
$ go install sigs.k8s.io/kind@v0.23.0
```

```bash
$ cat > kind-cluster-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: <name>   # here
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

```bash
# ingress
$ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
$ kubectl patch daemonsets -n projectcontour envoy -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/control-plane","operator":"Equal","effect":"NoSchedule"},{"key":"node-role.kubernetes.io/master","operator":"Equal","effect":"NoSchedule"}]}}}}'
```

```bash
# mgmt
$ kind get nodes --name istio-sidecar
$ kind get kubeconfig --name istio-sidecar
$ kind delete cluster --name <name>
```

### [k3s](https://k3s.io/)



### [kubspray](https://kubespray.io/#/)

