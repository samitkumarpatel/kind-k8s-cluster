## multi-node-cluster

**Create a cluster**

```sh
kind create cluster --config=$(pwd)/cluster.yml --kubeconfig=$(pwd)/kubeconfig
```

**Install metal-lb to gain LoadBalancer capability**

```sh
k --kubeconfig=$(pwd)/kubeconfig apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml

k --kubeconfig=$(pwd)/kubeconfig -n metallb-system get pods

# Define LoadBalancer ip pool
k --kubeconfig=$(pwd)/kubeconfig apply -f metallb-ipaddress-pool.yml
```
**Test the LoadBalancer capability**

```sh
k --kubeconfig=$(pwd)/kubeconfig run nginx --image=nginx --port=80                                                                                       

k --kubeconfig=$(pwd)/kubeconfig expose pod nginx --port=8080 --target-port=80 --type=LoadBalancer --name=nginx 

k --kubeconfig=$(pwd)/kubeconfig get svc # pick the external IP and hit the service from browser or from any command line tools.
```

**Install nginx Ingress controller[kubernetes contributed]**

[nginx ingress](https://kubernetes.github.io/ingress-nginx/deploy/)

```sh
k --kubeconfig=$(pwd)/kubeconfig apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.2/deploy/static/provider/cloud/deploy.yaml


k --kubeconfig=$(pwd)/kubeconfig wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```