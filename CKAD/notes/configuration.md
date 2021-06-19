#Defining and Consuming Configuration Data (cm)

#Creating a ConfigMap

## Creating a ConfigMap command
`kubectl create configmap <NAME> [--from-literar=key-value
                                |--from-env-file=<filename>
                                |--from-file=<filename>
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
Note section **envFrom**
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