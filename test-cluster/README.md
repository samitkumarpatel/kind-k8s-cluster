## test-cluster

**Create Cluster**

```sh
kind create cluster --config=$(pwd)/cluster.yml --kubeconfig=$(pwd)/kubeconfig

kubectl --kubeconfig kubeconfig get nodes -o wide
```
> This will add a `kubeconfig` file in the test-cluster folder, which can be use to interect with the cluster.

**Install Ingress Controllr**

```sh
 kubectl --kubeconfig kubeconfig apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/baremetal/deploy.yaml

```
> This will work but , it's not gonna run on control-plain and If Ingress controller is not gonna run in control plain , the expose port 80 and 443 will not give the necessary details.

```sh

```
