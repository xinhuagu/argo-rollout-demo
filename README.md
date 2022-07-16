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

### ArgoCD 

```
kubectl port-forward svc/argocd-server -n argocd 8081:443
