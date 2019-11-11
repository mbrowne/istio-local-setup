# istio-local-setup

Local minimal istio setup ready for apps to be installed and smoke tested. You can replace `helloworld` with any app you wish.

## Prerequisites

- Docker (tested with 8GB of RAM / 4 CPU's assigned)
- Kubernetes
  - You may use any Kubernetes setup you have e.g. MiniKube, Docker or even real hardware. Up to you. `kubectl config get-contexts` must point to your Kubernetes cluster. Here is a [tested configuration using Kubernetes using docker only and WSL on Windows](https://www.opvizor.com/combine-docker-kubernetes-and-windows-wsl)
- Helm

## Commands

```bash
# Download istio
curl -L https://github.com/istio/istio/releases/download/1.3.3/istio-1.3.3-linux.tar.gz | tar xz
cd istio-1.3.3

# Connect to local cluster in cat ~/.kube/config
kubectl config use-context docker-desktop

# Initialize helm / tiller in the cluster
helm init

# Install istio bootstrap
helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system

# Check it's loaded should say 23
kubectl get crds | grep 'istio.io' | wc -l

# Install minimal istio
helm install install/kubernetes/helm/istio \
    --name istio \
    --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-minimal.yaml \
    --set gateways.istio-ingressgateway.type=NodePort \
    --set gateways.enabled=true

# Check all services deployed
kubectl get svc -n istio-system

# Check all pods are running
kubectl get pods -n istio-system

# Check issues e.g. out of memory
#kubectl describe pods istio-pilot-86b95b7b74-tgrbv -n istio-system

# Enable automatic injection on the default namespace
#kubectl label namespace default istio-injection=enabled
#kubectl get namespace -L istio-injection

# Install the hello world chart and it's routing
helm install charts/istio-local-setup/ -n istio-local-setup --debug --dry-run

# Access the istio gateway
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```

Visit

http://localhost:8080/hello

and you should see

![](screenshot-success.png)

```bash
# Uninstall
helm del --purge istio-local-setup
```

helm install --name traefik --namespace default \
 --set dashboard.enabled=true,dashboard.auth.basic.admin='$apr1$n5r7oJvl\$NydaAb2oe.IudnNy4dCz3/' \
 stable/traefik
https://www.htaccesstools.com/htpasswd-generator/

# admin

# password
