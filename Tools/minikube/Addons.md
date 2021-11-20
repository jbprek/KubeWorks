# metrics-server installation
````bash
minikube addons enable metrics-server
# From https://www.educative.io/courses/advanced-kubernetes-techniques/qVYQ5g9pwAk
kubectl -n kube-system rollout status deployment metrics-server
```