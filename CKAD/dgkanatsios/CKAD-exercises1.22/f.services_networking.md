# Services and Networking (13%)

## Topics
- Demonstrate basic understanding of NetworkPolicies
- Provide and troubleshoot access to applications via services
- Use Ingress rules to expose applications

## Contents 

- [Services](#svc)
- [Nework Policy](#netpol)
- [Ingress](#ingress)
- [Record](#record)

## <a name="svc">Services</a>
### Setup

### S1 ClusterIP DNS
In Namespace s1

1. Create a service on top of a deployment named nginx with image nginx with 3 replicas and expose its port 80. Service should be named nginx-svc
2. Expose the deployment
3. Confirm that ClusterIP has been created. Also check endpoints
4. Get service's ClusterIP,
5. Create a temp busybox pod and 'hit' that ClusterIP with wget
6. Check from within the cluster (not from a pod) the service
7. Create a temp busybox pod and 'hit' the service using FQDN from Core-DNS.
8. 
<details><summary>show</summary>
<p>

```bash
#1 
kubectl -n s1 create deployment nginx --image nginx --replicas 3
# check deployment replica sets
kubectl get all -n s1
# Expose dep as a service
kubectl -n s1  expose deployment.apps/nginx --port 80 --name nginx-svc

#2
# Displays the full details including endpoints
kubectl -n s1  describe  svc nginx-svc
OR
# Display the details without endpoints
kubectl get svc nginx-svc # services
# get endpoint details
kubectl get ep # endpoints

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



### S2 NodePort
1.Convert Service from (S1) to NodePort for the same service
2. Find the NodePort port. 
3. Hit service using Node's IP. 

<details><summary>show</summary>
<p>

```bash
#1
kubectl -n s1 edit svc nginx
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
kubectl -n s1 patch svc nginx -p '{"spec":{"type":"NodePort"}}' 
```

```bash
# 2
kubectl -n s1 get svc nginx
OR
kubectl -n s1 describe svc nginx
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


### (S3) Service Misconfiguration
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


### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
kubectl label deployment foo --overwrite app=foo
```
</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

<details><summary>show</summary>
<p>


```bash
kubectl get pods -l app=foo -o wide # 'wide' will show pod IPs
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh
wget -O- POD_IP:8080 # do not try with pod name, will not work
# try hitting all IPs to confirm that hostname is different
exit
# or
kubectl get po -o wide -l app=foo | awk '{print $6}' | grep -v IP | xargs -L1 -I '{}' kubectl run --rm -ti tmp --restart=Never --image=busybox -- wget -O- http://\{\}:8080
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>


```bash
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo # you will see ClusterIP as well as port 6262
kubectl get endpoints foo # you will see the IPs of the three replica nodes, listening on port 8080
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>
<p>

```bash
kubectl get svc # get the foo service ClusterIP
kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
wget -O- foo:6262 # DNS works! run it many times, you'll see different pods responding
wget -O- SERVICE_CLUSTER_IP:6262 # ClusterIP works as well
# you can also kubectl logs on deployment pods to see the container logs
kubectl delete svc foo
kubectl delete deploy foo
```

</p>
</details>

## <a name="ingress">Ingress</a>

## <a name="netpol">Network Policy</a>
### NP1 Network Policy
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


### NP2 Network Policy CKAD Task weight: 9%

In Namespace venus you'll find two Deployments named hello and frontend. Both Deployments are exposed inside the cluster using Services. 
Create a NetworkPolicy named np1 which restricts outgoing tcp connections from Deployment frontend and only allows those going to Deployment hello.
Make sure the NetworkPolicy still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

## TODO Config map exercice create backend using configmap for proxy
Test using: wget www.google.com and wget hello:80 from a Pod of Deployment frontend.


#### Setup

Example from : https://kubernetes.io/docs/tasks/access-application-cluster/connecting-frontend-backend/


````bash

# Create backend deploy and service
kubectl create ns venus

````
````bash
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

# Test
kubectl -n venus run bus --image busybox -i $rm -- wget -O- hello:80 --timeout=2

````

`````bash

cat << EOF  > frontend.yaml   
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: hello
      tier: frontend
      track: stable
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        tier: frontend
        track: stable
    spec:
      containers:
        - name: nginx
          image: "gcr.io/google-samples/hello-frontend:1.0"
          lifecycle:
            preStop:
              exec:
                command: ["/usr/sbin/nginx","-s","quit"]  
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: hello
    tier: frontend
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 80
    nodePort: 32100
  type: NodePort 
EOF         

kubectl -n venus apply -f frontend.yaml

# Test
 curl localhost:32100 # From a node
kubectl -n venus run bus --image busybox -i $rm -- wget -O- frontend:80 --timeout=2
`````


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
      tier: frontend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: backend
      ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53
EOF

kubectl apply -f np1.yaml
````


</p>
</details>

