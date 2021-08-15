````bash
# Create namespace
kubectl create namespace labels
# Set namespace as the default for the current context
kubectl config set-context $(kubectl config current-context) --namespace=labels

#2. Create four Pods declared in the following multi-resource manifest file (pod-labels.yaml):
# Write the manifest file
cat << 'EOF' > pod-labels.yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-frontend
  namespace: labels # ceclare namespace in metadata 
  labels: # labels mapping in metadata
    color: red
    tier: frontend
  annotations: # Example annotation
    Lab: Kubernetes Pod Design for Application Developers
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: green-frontend
  namespace: labels
  labels:
    color: green
    tier: frontend
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: red-backend
  namespace: labels
  labels:
    color: red
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
---
apiVersion: v1
kind: Pod
metadata:
  name: blue-backend
  namespace: labels
  labels:
    color: blue
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
EOF
# Create the Pods
kubectl create -f pod-labels.yaml

## 3. se the -L (or --label-columns) kubectl get option to display columns for both labels:
kubectl get pods -L color,tier


## 4. Use the -l (or --selector) option to select all Pods with a color label:

kubectl get pods -L color,tier -l color

## 5. Select all Pods that do not have a color label:
kubectl get pods -L color,tier -l '!color'


## 6. Select all Pods that have the color red:
kubectl get pods -L color,tier -l 'color=red'

##7. Select all Pods that have the color red and are not in frontend tier:
kubectl get pods -L color,tier -l 'color=red,tier!=frontend'

##8. Select all Pods with green or blue color:
kubectl get pods -L color,tier -l 'color in (blue,green)'
9. Use the describe command to display the annotation for the red-frontend Pod:

kubectl describe pod red-frontend | grep Annotations


# The describe command is a convenient way to display annotations. 
# You can also set the output option (-o) of get to yaml to view all the fields of resources, including annotations,
# e.g. kubectl get pod red-frontend -o yaml.

 

## 10. Remove the Pod's Lab annotation and verify it no longer exists:

kubectl annotate pod red-frontend Lab- &&
kubectl describe pod red-frontend | grep Annotations -A 2
alt

# Only annotations related to the cluster’s network plugin remain. The annotate command can be used to add/remove/update annotations. 
# You add a dash (-) after the annotation key to remove the annotation.
# You can do the same with the kubectl label command when you need to remove a label.


## 11. Review the Examples in the annotate help:

kubectl annotate --help

## 12. Review the Examples in the label help:

kubectl label --help
alt

# The annotate and label commands are analogous. As the differences in the examples highlight, 
# annotations have less restrictions on their values. For example, label values cannot have spaces. 

````