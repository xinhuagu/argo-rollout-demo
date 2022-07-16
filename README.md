# Argo-rollout-demo

This DEMO demonstrates how to use Argo Workflow , Argo CD and Argo Rollout to create a complete Microservice CI/CD workflow with different Rollout Strategies (Canary/Blue-Green) in a kubernetes Cluster. 

Tech Stack:
- Kubernetes
- Quarkus
- Docker
- Tilt.dev
- Kaniko 
- Argo Workflow
- Argo CD
- Argo Rollout

![Alt text](/img/rollout-demo-process.jpeg?raw=true "Argo Rollout DEMO")


# Preparing the enviroment
## Local Kubernetes Cluster
Using Rancher desktop or docker desktop to start a local cluster

## Install metric server

Download components.yaml 
````
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
````
Add one line "--kubelet-insecure-tls" in "- args" 
````
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
 	- --kubelet-insecure-tls
````

````
kubectl apply -f components.yaml
````
## Install Argo Rollout

```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

Open Argo rollout dashboard

```
kubectl argo rollouts dashboard
```


## Argo Workflow server

```
argo server --auth-mode=server
```

### Install ArgoCD 
````
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
````
install argocd cli
````
brew install argocd
````


Open argocd ui
```
kubectl port-forward svc/argocd-server -n argocd 8081:443
````

Login and update admin credential
````
## get old password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
## login
export ARGOCD_OPTS='--port-forward-namespace argocd'
argocd login localhost:8081
## update password : Geheim123geheim
argocd account update-password 
````
### Install Argo workflow
````
kubectl create ns argo

kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml
````
install argo cli
````
brew install argo
````
start argo workflow ui
````
argo server --auth-mode=server
````

### Config the App enviroment

#### Install Ingress Controller
````
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
````

#### Create secret for docker.io
````
DOCKER_USERNAME=<username>
DOCKER_PASSWORD=<password>
DOCKER_SERVER=https://index.docker.io/v1/
kubectl create secret docker-registry regcred -n argo \
    --docker-server=${DOCKER_SERVER} \
    --docker-username=${DOCKER_USERNAME} \
    --docker-password=${DOCKER_PASSWORD}
````

#### create pvc for building process
````
kubectl apply -f ./development/kanikovolume.yaml
kubectl apply -f ./development/local-persistence.yaml
kubectl apply -f ./development/m2volume.yaml
````

# Buiding the project
start the buiding process ( make sure the tag uses right version number)
````
argo submit ./deployment/github.yaml -n argo -p tag=0.0.18
````

