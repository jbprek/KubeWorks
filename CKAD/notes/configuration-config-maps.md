#Defining and Consuming Configuration Data (cm)

# K8S doc
- [Kubernetes Documentation->Tasks->Configure Pods and Containers->Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
#Creating a ConfigMap

## Creating a ConfigMap command
`kubectl create configmap <NAME> [--from-literar=key-value
|--from-env-file=<filename>
|--from-file=<filename> | |--from-file=<key>=<filename> |
|--from-file=<dir-name>]`

## Creating a ConfigMap declaratively

```yaml
# backend-config.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  database_url: jdbc:postgresql://localhost/test
  user: fred
```

$ kubectl apply -f backend-config.yml

## Getting ConfigMap Info
### Get configuration maps
```
$ kubectl get cm 
```
### Get information about a configuration map
```
$ kubectl describe cm backend-config
# export in yaml format
$ kubectl get cm backend-config -oyaml
```

## Consuming/Using config map in a pod
Note section **envFrom** keywords **envFrom, configMapRef**
````yaml
#  configured-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    envFrom:
    - configMapRef:
        name: backend-config
````

```
kubectl apply -f configured-pod.yml
```


## Reassigning environment variable keys for ConfigMap entries
Note section **env** keywords **env, valueFrom, configMapRef**

````yaml
#  configured-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    env:
      - name: DATABASE_URL
        valueFrom:
          configMapKeyRef:
            name: backend-config
            key: database_url
      - name: USERNAME
        valueFrom:
          configMapKeyRef:
            name: backend-config
            key: user
````

```
kubectl apply -f configured-pod.yml
```

## Mounting a ConfigMap as Volume
Note sections **spec.containers.volumeMount, spec.volumes** 

````yaml
#  configured-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
    - image: nginx:1.19.0
      name: app
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: backend-config
````

# Exercises

- Create a configmap named config with values foo=lala,foo2=lolo
````bash
#create
kubectl create cm config-lit --from-literal=foo=lala --from-literal=foo2=lolo
# inspect 
kubectl describe cm config-lit

>> Output   
Name:         config-lit
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
foo:
----
lala
foo2:
----
lolo
Events:  <none>
Events:  <none>  

# inspect 1
kubectl get cm config-lit 
# inspect 2
kubectl get cm config-lit -o yaml

# delete
kubectl delete cm config-lit
````

- Create and display a configmap from a file
````bash
#create file 
echo -e "foo=1\nfoe=2" > config.txt
kubectl create cm config-file --from-file=config.txt
# inspect 
kubectl describe cm config-file

>> Output
Name:         config-file
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
config.txt:
----
foo=1
foe=2

Events:  <none>


````
- Create and display a configmap from a .env file
````bash
#create file 
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
kubectl create cm config-env --from-env-file=config.txt
# inspect 
kubectl describe cm config-env

>> Output
Name:         config-env
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
foo:
----
1
foe:
----
2
Events:  <none>
````

- Create and display a configmap from a file, giving the key 'special'
````bash
#create file 
echo -e "var3=val3\nvar4=val4" > config-key.txt
kubectl create cm config-file-key --from-env-file=special=config-key.txt
# inspect 
kubectl describe cm config-file-key

>> Output
Name:         config-file-key
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
special:
----
var3=val3
var4=val4
  

````
- Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

Search example in docs
````yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
````
````bash
#create Create a configMap called 'options' with the value var5=val5. 
echo -e "var3=val3\nvar4=val4" > config-key.txt
kubectl create cm options --from-literal=var5=val5
# inspect configMap
kubectl describe cm options
# Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'
kubectl run ngix-option --image nginx --restart=Never --dry-run=client -o yaml > ngix-option.yaml
less ngix-option.yaml

````
Output
````yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ngix-option
  name: ngix-option
spec:
  containers:
  - image: nginx
    name: ngix-option
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
````

- Edit yaml
````yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ngix-option
  name: ngix-option
spec:
  containers:
  - image: nginx
    name: ngix-option
    env:
      - name: option
        valueFrom:
          configMapKeyRef:
            name: options
            key: var5
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
````
- create pod
````
kubectl apply -f  ngix-option.yaml
````
# TODO Questions

- What happens when a config map in use is deleted

