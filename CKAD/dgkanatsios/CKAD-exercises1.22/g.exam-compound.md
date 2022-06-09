# Exam Compound


## clean-up:
```bash
# E1
rm -f /opt/ckad/moon/*

kubectl delete ns moon

# E2
rm -f /opt/ckad/neptune/*
kubectl delete ns moon
```

## Setup:
```bash
# E1
mkdir -p /opt/ckad/moon/

kubectl create ns moon

cat << EOF > /opt/ckad/moon/webserver-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    id: webserver
  name: webserver
  namespace: moon
spec:
  containers:
  - image: nginx
    name: webserver
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

kubectl apply -f /opt/ckad/moon/webserver-pod.yaml

# E2
mkdir -p /opt/ckad/neptune/
kubectl create ns neptune

cat << EOF > /opt/ckad/neptune/nginx-114.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-114
    service: webserver
  name: nginx-114
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-114
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        service: webserver
        app: nginx-114
    spec:
      containers:
      - image: nginx:1.14
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
EOF

kubectl -n neptune apply -f /opt/ckad/neptune/nginx-114.yaml
kubectl -n neptune expose deploy nginx-114 --port=80
```


# E1 Pod to deployment, Security Context 6 minutes
In namespace moon there's a pod called webserver
1. Transform the pod into a deployment named webserver with 3 replicas, save the manifest as /opt/ckad/moon/webserver.yaml 
2. In addition, the new Deployment should set allowPrivilegeEscalation: false and privileged: false for the security context on container level.


<details><summary>show</summary>
<p>

</p>
</details>


# E2 Canary deployment
In namespace neptune there is a service running exposing deployment nginx-114, called nginx-svc
1. Create a canary deployment updating the image to nginx:1.18
2. Test then canary deployment 
3. Once you're confident that the canary instance is running properly
    1. Scale Up the canary deployment to 3 instances
    2. Scale down the original deployment to 0

<details><summary>show</summary>
<p>

```bash
# 1.1
kubectl create ns s5
kubectl create deployment nginx-114 --image=nginx:1.14 --port=80 --replicas=2 -n s5 $do > d1.yml
```



````yaml
# 1.2 Edit d1.yml file as follows:
# Add labels to pod and deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-114
    service: webserver
  name: nginx-114
  namespace: s3
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-114
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        service: webserver
        app: nginx-114
    spec:
      containers:
      - image: nginx:1.14
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
````
```bash
# 1.3
kubectl apply -f d1.yml
```

```bash
# 2
kubectl expose deployment nginx-114 --port=80 --selector=service=webserver --name nginx-svc
```

````yaml
# 3.1 copy d1.yml file into d2.yml and edit as follows:
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-119
    service: webserver
  name: nginx-119
  namespace: s5
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-119
  strategy: {}
  template:
    metadata:
      labels:
        service: webserver
        app: nginx-119
    spec:
      containers:
        - image: nginx:1.19
          name: nginx
          ports:
            - containerPort: 80
          resources: {}
status: {}

````
```bash
# 3.2
kubectl apply -f d2.yml
```

```
# 4
# 4.1 Inspect pods deploy svc
$ kubectl -n s5 get all

NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
pod/nginx-114-86f99b7c4-zlr4d    1/1     Running   0          21m   10.1.230.179   ubuk8s   <none>           <none>
pod/nginx-114-86f99b7c4-5sc9f    1/1     Running   0          21m   10.1.230.178   ubuk8s   <none>           <none>
pod/nginx-119-55494f98f4-kmcrn   1/1     Running   0          50s   10.1.230.180   ubuk8s   <none>           <none>

NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/nginx-svc   ClusterIP   10.152.183.165   <none>        80/TCP    8m30s   service=webserver

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
deployment.apps/nginx-114   2/2     2            2           21m   nginx        nginx:1.14   app=nginx-114
deployment.apps/nginx-119   1/1     1            1           50s   nginx        nginx:1.19   app=nginx-119

NAME                                   DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES       SELECTOR
replicaset.apps/nginx-114-86f99b7c4    2         2         2       21m   nginx        nginx:1.14   app=nginx-114,pod-template-hash=86f99b7c4
replicaset.apps/nginx-119-55494f98f4   1         1         1       50s   nginx        nginx:1.19   app=nginx-119,pod-template-hash=55494f98f4

# 4.1 Inspect endpoints
$ kubectl -n s5 get ep
NAME        ENDPOINTS                                         AGE
nginx-svc   10.1.230.178:80,10.1.230.179:80,10.1.230.180:80   8m43s

```
We can see that the new pod ip is included in the service's endpoints.



</p>
</details>

