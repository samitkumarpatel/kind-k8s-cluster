Example of metallb, Ingress

```sh
kind create cluster --config=$(pwd)/cluster.yml --kubeconfig=$(pwd)/kubeconfig

k --kubeconfig=$(pwd)/kubeconfig apply -f tools/metallb-native.yaml
k --kubeconfig=$(pwd)/kubeconfig -n metallb-system get pods

k --kubeconfig=$(pwd)/kubeconfig apply -f tools/metallb-ipaddress-pool.yml

k --kubeconfig=$(pwd)/kubeconfig apply -f tools/nginx-ingress-kind.yml
k --kubeconfig=$(pwd)/kubeconfig -n ingress-nginx get all

k --kubeconfig=$(pwd)/kubeconfig apply -f ingress-example.yaml

kind delete cluster --name=kind

```