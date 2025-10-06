# kind with metallb

[Reference Guide](https://medium.com/@tylerauerbeck/metallb-and-kind-loads-balanced-locally-1992d60111d8).

### Create single node cluster.

```sh
kind create cluster --kubeconfig=$(pwd)/kubeconfig --config=$(pwd)/cluster.yml

#OR

kind create cluster --name=k8s-lab --kubeconfig=$(pwd)/kubeconfig 
```

### Install Metallb

```sh
kubectl apply --kubeconfig kubeconfig -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml  
```

### Install IPAddressPool

How you know what would be your expected Ip address range ?

```sh
kubectl get nodes -o wide
# Get the InternalIp and try to ping

ping 192.16.0.2
```
If the ping has success reply , you are good to go with that range, like below or else troubleshoot why the ping doesnot work.

```sh
cat <<EOF | kubectl apply --kubeconfig kubeconfig -f -
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: demo-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.18.255.1-172.18.255.25
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: demo-advertisement
  namespace: metallb-system
EOF

```

### Install LoadBalancer

- pod & Services

```sh
cat <<EOF | kubectl apply --kubeconfig kubeconfig -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels:
        app: echo
    spec:
      containers:
        - name: echo
          image: hashicorp/http-echo
          args:
            - -listen=:8080
            - -text="hello there"
          ports:
            - name: http
              containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    metallb.universe.tf/address-pool: demo-pool 
  name: echo
spec:
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
  selector:
    app: echo
  type: LoadBalancer
EOF

```
- Service: Load bancer

`metallb.universe.tf/address-pool: demo-pool` :This name to be from `IPAddressPool`.

> This Loadbalancer should have an external Ip now - Use this to access access