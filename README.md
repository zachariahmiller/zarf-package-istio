# Testing istio deployment via zarf

## Create k3d cluster

Use util/k3d-calico-mlb.yaml and then properly configure calico and metallb or use util/k3d.yaml to provision your cluster via k3d cluster create -c <file>

Note: before creating your cluster update the tlssan with your hosts ip "--tls-san=192.168.0.196"

## Build Zarf package

This assumes you have zarf installed, if you dont then you need to install it.

```bash
zarf p c . --confirm
```

## Initialize cluster and deploy zarf package

```bash
zarf init --components="git-server" -a amd64
zarf package deploy zarf-package-istio-amd64-0.0.1.tar.zst --confirm
```

## Deploy Gateway CR that isnt in zarf.yaml yet

```bash
zarf tools kubectl apply -f gateway-and-cert.yaml
```

## Add injection label on default NS 

```bash
kubectl label namespace default istio-injection=enabled --overwrite
```

## Deploy Test app

```bash
zarf tools kubectl apply -f test.yaml
```

## Test

```bash
export INGRESS_NS="istio-system"
export INGRESS_NAME="admin-ingressgateway"
export INGRESS_HOST=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')

curl -vvv -I -HHost:httpbin.bigbang.dev "http://$INGRESS_HOST:$INGRESS_PORT/status/200"


## Check logs
kubectl logs --all-containers <admin ingress gateway pod> -n istio-system -f
```

## :cry: why no working?

## :persevere: and try again doing something different
