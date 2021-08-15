#Introduction
Kubernetes uses ServiceAccounts as a mechanism for providing Pods with an identity in the cluster. Pods can authenticate using ServiceAccounts and gain access to APIs that the ServiceAccount has been granted. Your cluster administrator can create specific roles that grant access to APIs and bind the roles to ServiceAccounts. This is referred to as role-based access control (RBAC). Pods can then declare a ServiceAccount in their specification to gain the access associated with the ServiceAccount's role. As an example, you could use a ServiceAccount to grant a Pod access to the GET Pod API to allow the Pod to get the details of other Pods. This lab step focuses on ServiceAccounts and not the roles that are used to grant access to Kubernetes APIs that would be configured by a Kubernetes administrator.
````bash
# 0. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

#1. 1. Create a Namespace for the resources you'll create in this Lab Step and change your default kubectl context to use the Namespace:
# Create namespace
kubectl create namespace serviceaccounts
# Set namespace as the default for the current context
kubectl config set-context --current --namespace=serviceaccounts
#Verify
kubectl config view | grep namespace:

#2. Get the ServiceAccounts in the Namespace:
kubectl get serviceaccounts
  
  
#3. Create a Pod and get its YAML manifest:  
kubectl run default-pod --image=mongo:4.0.6
kubectl get pod default-pod -o yaml | more

#4. Create a new ServiceAccount:
kubectl create serviceaccount app-sa

#5. Create a new Pod that uses the custom ServiceAccount:
cat << 'EOF' > pod-custom-sa.yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-sa-pod 
spec:
  containers:
  - image: mongo:4.0.6
    name: mongodb
  serviceAccount: app-sa
EOF
kubectl create -f pod-custom-sa.yaml
# Consider --serviceaccount=app-sa with kubectl run, to create manifest
kubectl run default-pod --image=mongo:4.0.6 --serviceaccount=app-sa --dry-run=client -o yaml

#6. Get the Pod's YAML manifest:
kubectl get pod custom-sa-pod -o yaml | less
````