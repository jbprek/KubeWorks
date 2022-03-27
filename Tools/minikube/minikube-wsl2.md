#Does no work
# Setup Local Registry

## Enable Registry addon
Modified instructions from [minikube docs](https://minikube.sigs.k8s.io/docs/handbook/registry/)

````bash
minikube addons enable registry

# Port for registry 53926 instead of 50000

#Check catalog
curl http://localhost:53926/v2/_catalog

#Use kubectl port-forward to map your local workstation to the minikube vm

kubectl port-forward --namespace kube-system service/registry 53926:80


docker run --rm -it --network=host alpine ash -c \
"apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:host.docker.internal:53926"


````