# Exam Compound


## Clean-up:
<details><summary>show</summary>
<p>

```bash
# E1
rm -f /opt/ckad/moon/*

kubectl delete ns moon

# E2
rm -f /opt/ckad/neptune/*
kubectl delete ns neptune

# E3
rm -f /opt/ckad/minerva/*
kubectl delete ns minerva
```
</p>
</details>

## Setup:
<details><summary>show</summary>
<p>


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
    release: nginx-114
    service: nginx-svc
  name: nginx-114
spec:
  replicas: 3
  selector:
    matchLabels:
      release: nginx-114
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        service: nginx-svc
        release: nginx-114
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
kubectl -n neptune expose deploy nginx-114 --port=80 --name=nginx-svc --selector=service=nginx-svc

# E3
mkdir -p /opt/ckad/minerva/
kubectl create ns minerva

cat << EOF > /opt/ckad/minerva/home-page1.html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to deploy version 1</title>
    <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
</head>
<body>
<h1>Welcome to deploy version 1</h1>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
EOF

cat << EOF > /opt/ckad/minerva/home-page2.html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to deploy version 1</title>
    <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
</head>
<body>
<h1>Welcome to deploy version 1</h1>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
EOF
```


</p>
</details>


# E1 Pod to deployment, Security Context 6 minutes
In namespace moon there's a pod called webserver
1. Transform the pod into a deployment named webserver with 3 replicas, save the manifest as /opt/ckad/moon/webserver.yaml 
2. In addition, the new Deployment should set allowPrivilegeEscalation: false and privileged: false for the security context on container level.


<details><summary>show</summary>
<p>

</p>
</details>


# E2 Canary deployment 10 minutes
In namespace neptune there is a service running exposing deployment nginx-114, called nginx-svc
1. Create a canary deployment updating the image to nginx:1.18
2. Test then canary deployment 
3. Once you're confident that the canary instance is running properly
    1. Scale Up the canary deployment to 3 instances
    2. Scale down the original deployment to 0

<details><summary>show</summary>
<p>


</p>
</details>


# E3 Config map 5 minutes
Create a config map in namespace minerva from file /opt/ckad/minerva/home-page1.html
Create a pod name cmweb in namespace minerva mounting this file as /usr/share/nginx/html/index.html using as image nginx:latest
Verify that the file is served as the root page of the pod.

<details><summary>show</summary>
<p>


</p>
</details>


