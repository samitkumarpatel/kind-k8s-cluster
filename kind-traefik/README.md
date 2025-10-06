# kind-traefik

[Reference guide from Traefik documentation](https://doc.traefik.io/traefik/getting-started/kubernetes/).

### Cluster Creation

```sh
kind create cluster --name=k8s-lab --kubeconfig=$(pwd)/kubeconfig
export KUBECONFIG=$(pwd)/kubeconfig
```

### Metallb Installation

```sh
kubectl apply --kubeconfig kubeconfig -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml

k get nodes -o wide

```
> Get the InternalIp and try to ping, If it can be ping , define a Iprange to be used for Metallb IPAddressPool , Like below::

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

### Traefik Installation

```sh
helm repo add traefik https://traefik.github.io/charts
helm repo update

helm install traefik traefik/traefik -f values.yml --wait
```
**Test If dashboard is accessable**

```sh
curl -sv --resolve dashboard.localhost:80:172.18.255.2 http://dashboard.localhost
```


### Test IngressRoute and HTTPRoute CRD for a whoami service.

```sh
cat <<EOF | kubectl apply --kubeconfig kubeconfig -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  replicas: 2
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
    - port: 80
  selector:
    app: whoami
---
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: whoami
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`whoami.localhost`)
      kind: Rule
      services:
        - name: whoami
          port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: whoami
spec:
  parentRefs:
    - name: traefik-gateway
  #hostnames:
  #  - "whoami-gatewayapi.localhost"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: whoami
          port: 80
```

**Test If whoami service is accessable with HTTPRoute and with IngressRoute**

```sh
curl -sv --resolve whoami.localhost:80:172.18.255.2 http://whoami.localhost

kubectl --kubeconfig kubeconfig get svc traefik
# Copy the Loadbalancer External Ip
# Open browser and access the Ip - You can see whoami service response on your browser.
```