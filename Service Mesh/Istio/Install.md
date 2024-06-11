## [Sidecar](https://istio.io/latest/docs/setup/install/)

### istioctl

```bash
# download istioctl
$ curl -L https://istio.io/downloadIstio | sh -

# add istioctl to PATH
$ cd istio-*/bin
$ export PATH=$PWD/bin:$PATH
```

```bash
# install
$ istioctl install --set profile=demo -y
```

```bash
# list avail profile to install
$ istioctl profile list

# diff profile
$ istioctl profile p1 p2

# dump profile
$ istioctl profile dump demo

# dump config
$ istioctl profile dump --config-path components.pilot demo
$ istioctl profile dump --config-path components.galley demo
$ istioctl profile dump --config-path components.citadel demo

# gen manifest
$ istioctl manifest generate > istio-manifest.yaml
```

```bash
# uninstall
$ istioctl uninstall --purge 
```

### Helm

```bash
# add repo
$ helm repo add istio https://istio-release.storage.googleapis.com/charts
$ helm repo update
```

```bash
# create ns
$ kubectl create ns istio-system
$ kubectl create ns istio-ingress

# install
$ helm install istio-base istio/base -n istio-system --set defaultRevision=default
$ helm install istiod istio/istiod -n istio-system --wait
$ helm install istio-ingress istio/gateway -n istio-ingress --wait

# chk
$ helm ls -n istio-system
```

```bash
# uninstall
$ helm delete istio-ingressgateway -n istio-ingress
$ helm delete istiod -n istio-system
$ helm delete istio-base -n istio-system

$ kubectl delete ns istio-ingress istio-ingress
```

## [Ambient](https://istio.io/latest/docs/ambient/install/)

### Helm