# kind-k8s-cluster

Build a Kubernetes playground on demand to test any kubernetes workload / Operator / ect.. as quickly as possible.

## Getting Started

### Install / Create a Cluster

> Make su re , You have `kind` command line tool Installed.

```sh
kind create cluster --kubeconfig $(pwd)/kubeconfig --name kind-playground
```
> The `kubeconfig` file has the kubernetes cluster config, which help you to get connect to the kind cluster and do all admin activity via kubectl command.


### Create several users (like Admin, Reader & Writer user) to deal with this cluster.


**user1::** read/view access to the cluster

```sh
#Create Certificate

openssl genrsa -out user1.key 2048 
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1"
openssl x509 -req -in user1.csr -CA $(pwd)/ca.crt -CAkey $(pwd)/ca.key -CAcreateserial -out user1.crt -days 30

#RoleBinding

kubectl --kubeconfig=$(pwd)/kubeconfig create rolebinding role-view --clusterrole=view --user=user1

#Create kubeconfig file specific to the user

kubectl config set-cluster kind-playground-cluster --certificate-authority=$(pwd)/ca.crt --embed-certs=true --server=https://127.0.0.1:59588 --kubeconfig=$(pwd)/user1-view-kubeconfig

kubectl config set-credentials user1 --client-certificate=$(pwd)/user1.crt --client-key=$(pwd)/user1.key --embed-certs=true --kubeconfig=$(pwd)/user1-view-kubeconfig

kubectl config set-context kind-playground-cluster-ctx --cluster=kind-playground-cluster --user=user1 --kubeconfig=$(pwd)/user1-view-kubeconfig

kubectl config use-context kind-playground-cluster-ctx --kubeconfig=$(pwd)/user1-view-kubeconfig

#Test
kubectl --kubeconfig user1-view-kubeconfig get pods #will work
kubectl --kubeconfig user1-view-kubeconfig get ns #will fail
kubectl --kubeconfig kubeconfig get nodes #will fail

kubectl --kubeconfig kubeconfig get nodes #will work
```
### LoadBaalncer - metal lb

[metal lb reference](https://medium.com/@tylerauerbeck/metallb-and-kind-loads-balanced-locally-1992d60111d8).
