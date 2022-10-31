# Services and Networking (13%)

## Topics
- Demonstrate basic understanding of NetworkPolicies
- Provide and troubleshoot access to applications via services
- Use Ingress rules to expose applications

## Contents 

- [Services](#svc)
- [Nework Policy](#netpol)
- [Ingress](#ingress)

## <a name="svc">Services</a>
### Setup

### SVC.1 ClusterIP DNS

In Namespace s1

1. Create a service on top of a deployment named nginx with image nginx with 3 replicas and expose its port 80. Service should be named nginx-svc
2. Expose the deployment
3. Confirm that ClusterIP has been created. Also check endpoints
4. Get service's ClusterIP,
5. Create a temp busybox pod and 'hit' that ClusterIP with wget
6. Check from within the cluster (not from a pod) the service
7. Create a temp busybox pod and 'hit' the service using FQDN from Core-DNS.

<details><summary>show</summary>
<p>

```bash
#1 
kubectl -n s1 create deployment nginx --image nginx --replicas 3
# check deployment replica sets
kubectl get all -n s1
# Expose dep as a service
kubectl -n s1  expose deployment.apps/nginx --port 80 --name nginx-svc-cip

#2
# Displays the full details including endpoints
kubectl -n s1  describe  svc nginx-svc
OR
# Display the details without endpoints
kubectl -n s1 get svc nginx-svc # services
# get endpoint details
kubectl -n s1 get ep # endpoints

#3
kubectl -n s1  get svc nginx # get the IP (something like 10.108.93.130)
kubectl -n s1  run busybox --rm --image=busybox -it --restart=Never -- sh
wget -O- IP:80
exit
#OR
CLUSTER-IP=$(kubectl -n s1 get svc nginx --template={{.spec.clusterIP}}) # get the IP (something like 10.108.93.130)

#5
kubectl run busybox --image=busybox -i -- wget -O- CLUSTER-IP --timeout 2
# Tip: --timeout is optional, but it helps to get answer more quickly when connection fails (in seconds vs minutes)

#6
# From within a terminal inside the cluster i.e minikube ssh
curl IP

#7
kubectl run bus1 $rm --image busybox -i -- wget -O- nginx-svc.s1.svc.cluster.local
```

</p>
</details>



### SVC.2 NodePort
1.Convert Service from (S1) to NodePort for the same service
2. Find the NodePort port. 
3. Hit service using Node's IP. 

<details><summary>show</summary>
<p>

```bash
#1
kubectl -n s1 edit svc nginx-svc
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    # NOTE you can specify the port below
    nodePort: 32100
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # change Cluster IP to nodeport
status:
  loadBalancer: {}
```

In vi write and exit :x

or

```bash
kubectl -n s1 patch svc svc nginx-svc -p '{"spec":{"type":"NodePort"}}' 
```

```bash
# 2
kubectl -n s1 get svc svc nginx-svc
OR
kubectl -n s1 describe svc svc nginx-svc
```


```bash
wget -O- NODE_IP:31931 # if you're using Kubernetes with Docker for Windows/Mac, try 127.0.0.1
#if you're using minikube, try minikube ip, then get the node ip such as 192.168.99.117
# from inside microK8s
curl localhost:32100
# from outside the cluster
curl ubu1.vm:32100
```

```bash
kubectl delete svc nginx # Deletes the service
kubectl delete deploy nginx # Deletes the pod
```
</p>
</details>


### SVC.2.1 NodePort

1. Drop  Service nginx-svc and create a new one with the same name using NodePort  3000
2. Hit service using Node's IP.

<details><summary>show</summary>
<p>

```bash
#1
kubectl -n s1 expose deployment.apps/nginx --port 80 --type=NodePort --name nginx-svc -o yaml --dry-run=client > nginx-svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    # NOTE you can specify the port below
    nodePort: 32100
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # change Cluster IP to nodeport
status:
  loadBalancer: {}
```

In vi write and exit :x

or

```bash
kubectl -n s1 patch svc svc nginx-svc -p '{"spec":{"type":"NodePort"}}' 
```

```bash
# 2
kubectl -n s1 get svc svc nginx-svc
OR
kubectl -n s1 describe svc svc nginx-svc
```


```bash
wget -O- NODE_IP:31931 # if you're using Kubernetes with Docker for Windows/Mac, try 127.0.0.1
#if you're using minikube, try minikube ip, then get the node ip such as 192.168.99.117
# from inside microK8s
curl localhost:32100
# from outside the cluster
curl ubu1.vm:32100
```

```bash
kubectl delete svc nginx # Deletes the service
kubectl delete deploy nginx # Deletes the pod
```
</p>
</details>



### SVC.3  Service Misconfiguration
There is a problem accessing the service nginx through curl
1. Setup
````bash
kubectl create namespace s3
kubectl -n s3 run nginx --image nginx --expose port 80
kubectl -n s3 label pod nginx run=mynginx --overwrite
````

2. Fix the problem in the service
3. Verify that we get the answer from the service
4. Show log entry from the pod
<details><summary>show</summary>
<p>

```bash
#2
kubectl get all -n s3
# We can see the cluster IP of the service that is not accessible through
curl CLUSTER-IP
# We can check the response of the pod from a 
kubectl run alpine --image nginx:alpine -- sh -c "sleep 7200"
kubectl exec alpine -it -- sh
# From the shell run curl POD-IP , shoud respond OK

# Endpoint check reveals no endpoint
kubectl -n s3 ep
# check labels of the pod
kubectl -n s3 get pods --show-labels
# check service selector
kubectl -n s3 describe svc nginx
# There is a mismatch in the selector of the service and the pod's label
kubectl -n s3 edit svc nginx

#3 
curl CLUSTER-IP
#4 
kubectl-n s3 logs pod POD-NAME 
```
</p>
</details>


### SVC.4 Deployment/Service 
In namespace s4
1.Create a deployment 
    1.called foo 
    2. using image 'jbprek/hello-from-ip'
    3. replicas. 
    5. Declare that containers in this pod will accept traffic on port 8080 
2. Test Deployment
   1. Get the pod IPs.
   2. Create a temp busybox pod and try hitting them on port 8080
3. Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints
4. Verify that each time there's a different hostname returned. 
5. Delete namespace to cleanup the cluster
 

<details><summary>show</summary>
<p>

```bash
# 1
kubectl create ns s4
kubectl -n s4 create deploy foo --image=jbprek/hello-from-ip --port=8080 --replicas=3

# 2
kubectl get pods -l app=foo -o wide # 'wide' will show pod IPs
kubectl run bus2 -n s4 --image busybox $rm -i -- wget -O- http://POD_IP:8080/hello

```

```bash
# 3
kubectl -n s4  expose deploy foo --port=6262 --target-port=8080
kubectl -n s4  get service foo # you will see ClusterIP as well as port 6262
kubectl -n s4 get ep foo # you will see the IPs of the three replica nodes, listening on port 8080
```

```bash
# 4 
kubectl delete ns s4
```
</p>
</details>

### SVC.5 Deployment/Service Expose service based on selector 
In namespace s5
This the backbone of Canary deployments

1. Create a deployment nginx-114 with nginx version 1.14 label deployment and pods labeled with 'service=webserver' and 2 replicas
2. Create a service exposing the above deploy named nginx-svc
3. Create a deployment nginx-119 with nginx version 1.19 label deployment and pods labeled with 'service=webserver' and 1 replica
4. Observe service and endpoints that are picking up 

9. Delete namespace to cleanup 


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


## <a name="ingress">Ingress</a>
### ING.1 Simple rule
### Setup
- Make sure an ingress addons is available
- make sure that IP of the cluster is accessible through DNS

```bash
kubectl create ns ing
kubectl -n ing create deployment nginx --image=nginx --port=80  --replicas 3
kubectl -n ing expose deployment nginx --port=8080 --target-port=80
kubectl create ns ing2
kubectl -n ing2 create deployment apache --image=httpd --port=80 --replicas 3
kubectl -n ing2 expose deployment apache --port=8080 --target-port=80
```

### Cleanup
```bash
kubectl -n ing svc nginx
kubectl -n ing delete deployment nginx
kubectl -n ing2 svc httpd
kubectl -n ing2 delete deployment apache
kubectl delete ns ing
kubectl delete ns ing2
```


There are 2 services in the cluster 
- one is exposed in ing namespace named nginx
- the second in ing2 namespace named apache

Expose through ingress 
- nginx under external url /hello
- apache under external url /goodbye

<details><summary>show</summary>
<p>

```bash
# 1

kubectl -n ing create ingress simple --rule="ubu1.info/hello=nginx:8080"

```

```bash
# 3
kubectl -n s4  expose deploy foo --port=6262 --target-port=8080
kubectl -n s4  get service foo # you will see ClusterIP as well as port 6262
kubectl -n s4 get ep foo # you will see the IPs of the three replica nodes, listening on port 8080
```

```bash
# 4 
kubectl delete ns s4
```
</p>
</details>


## <a name="netpol">Network Policy</a>
### NP.1 Network Policy
Create an nginx deployment of 2 replicas, in namespace np1 expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

<details><summary>show</summary>
<p>

```bash
kubectl -n np1 create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80

kubectl -n np1 describe svc nginx # see the 'app=nginx' selector for the pods
# or
kubectl -n np1 get svc nginx -o yaml

vi policy.yaml
```

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # pick a name
  namespace: np1
spec:
  podSelector:
    matchLabels:
      app: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: granted
```

```bash
# Create the NetworkPolicy
kubectl create -f policy.yaml

# Check if the Network Policy has been created correctly
# make sure that your cluster's network provider supports Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # This should not work. --timeout is optional here. But it helps to get answer more quickly (in seconds vs minutes)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # This should be fine
```

</p>
</details>


### NP.2 Network Policy CKAD Task weight: 9%

Example (From)[ https://kubernetes.io/docs/tasks/access-application-cluster/connecting-frontend-backend/]

In Namespace venus you'll find two Deployments named hello and frontend. Both Deployments are exposed inside the cluster using Services. 
Create a NetworkPolicy named np1 which restricts outgoing tcp connections from Deployment front-end and only allows those going to Deployment hello.
Make sure the NetworkPolicy still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

## TODO Config map exercice create backend using configmap for proxy
Test using: wget www.google.com and wget hello:80 from a Pod of Deployment frontend.


#### Setup

````bash
kubectl create ns venus
# Backend service
cat << EOF  > backend.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  selector:
    matchLabels:
      app: hello
      tier: backend
      track: stable
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        tier: backend
        track: stable
    spec:
      containers:
        - name: hello
          image: "gcr.io/google-samples/hello-go-gke:1.0"
          ports:
            - name: http
              containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: http              
EOF

kubectl -n venus apply -f backend.yaml

# Create front-end 

cat << EOF  > front-end-nginx.conf
# The identifier Backend is internal to nginx, and used to name this specific upstream
upstream Backend {
    # hello is the internal DNS name used by the backend Service inside Kubernetes
    server hello;
}

server {
    listen 80;

    location / {
        # The following statement will proxy traffic to the upstream named Backend
        proxy_pass http://Backend;
    }
}
EOF

kubectl -n venus create cm front-end-cm --from-file=front-end-nginx.conf

cat << EOF > front-end.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-end
  name: front-end
  namespace: venus
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  strategy: { }
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
        - image: nginx:alpine
          name: front-end
          volumeMounts:
            - name: config-volume
              mountPath: /etc/nginx/conf.d
          resources: { }
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      volumes:
        - name: config-volume
          configMap:
            name: front-end-cm
EOF

kubectl -n venus apply -f front-end.yml

kubectl -n venus expose deployment.apps/front-end  --port=80
````


```bash
# Test backend
kubectl -n venus run bus --image busybox -i $rm -- wget -O- hello:80 --timeout 2
# Test front-end
kubectl -n venus run bus --image busybox -i $rm -- wget -O- front-end:80 --timeout 2
```


<details><summary>show</summary>
<p>

````bash
cat << EOF > np1.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np1
  namespace: venus
spec:
  podSelector:
    matchLabels:
      app: front-end
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: hello
# Please note the difference if in the below leading (-) in front of ports was missing
# Current rule is (target has label app:hello) OR ( ports in 53)
# With missing (-) is  (target has label app:hello) AND ( ports in 53)             
    - ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53
EOF

kubectl apply -f np1.yaml
````


</p>
</details>

