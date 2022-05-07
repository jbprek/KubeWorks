# Dashboard
## Port Forward 

````bash
kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443 --address 0.0.0.0
````
## Get Token for microk8s dashboard

````bash
token=$(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token

# One line - Clear token

token=$(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1) -o jsonpath='{.data.token}' | base64 -d) && echo -e "$token\n"
````

## Dashboard URL
https://192.168.0.172:10443

## Original Instructions

From : https://microk8s.io/docs/addon-dashboard

