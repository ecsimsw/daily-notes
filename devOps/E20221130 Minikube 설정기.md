## Minikube

Env : M1 Mac 

Install minikube v1.25.1 : 
```
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.25.1/minikube-darwin-arm64 && chmod +x minikube
```
```
sudo install minikube /usr/local/bin/minikube
```

```
minikube version
```

Start minikube with docker : 
```minikube start --driver=docker```

