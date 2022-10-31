# Kind - K8S Dashboard  

# 1. Deploy dashboard

````bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

````
# 2. Verify dashboard deployment

````bash
kubectl get pod -n kubernetes-dashboard
````

# 3.Create Service Account

````bash
kubectl create serviceaccount -n kubernetes-dashboard admin-user
kubectl create clusterrolebinding -n kubernetes-dashboard admin-user --clusterrole cluster-admin --serviceaccount=kubernetes-dashboard:admin-user

````

# 3.Create Service Account Token

````bash
echo $(kubectl -n kubernetes-dashboard create token admin-user)

````

# 4.Access Dashboard

````bash
$ kubectl proxy
````

Use the token from (3) to login:
[Kubernetes Login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)

# Notes
- Instructions from https://istio.io/latest/docs/setup/platform-setup/kind/

