## [minikube](https://minikube.sigs.k8s.io/docs/)

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



