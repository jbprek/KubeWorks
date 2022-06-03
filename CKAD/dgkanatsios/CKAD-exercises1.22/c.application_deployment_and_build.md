#Application  Deployment 20 %

## Curicullum
- Use Kubernetes primitives to implement common deployment strategies (e.g. blue/ green or canary)
- Understand Deployments and how to perform rolling updates
- Use the Helm package

## Topics
- [Labels & Annotations](#Labels)
- [Deployments](#Deployments)
- [Helm Management](#Helm)
- [Canary Blue/Green Deployments](#bluegreen)
- [Record](#record)

## TODO Blue/Green canary
- [Harness Article](https://harness.io/blog/blue-green-canary-deployment-strategies/)


## <a name="Labels">Labels & Annotations</a>

Remember -l -L options

### L1. Show all labels of the pods
Create 3 nginx pods with names nginx1, nginx2, nginx3

```bash
kubectl run nginx1 --image=nginx
kubectl run nginx2 --image=nginx
kubectl run nginx3 --image=nginx
```

Show all labels of the pods:

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### L2. Set label of all nginx pods to app=v1

<details><summary>show</summary>
<p>

```bash
kubectl label nginx{1..3} app=v1
```

</p>
</details>

### L3 Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
```

</p>
</details>

### L4 Get the label 'app' for the pods (show a column with APP labels)

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
# or
kubectl get po --label-columns=app
```

</p>
</details>

### L5 Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
# or
kubectl get po --selector=app=v2
```

</p>
</details>

### L6 Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -l app app-
```

</p>
</details>

### L7 Create a nignx pod named nginx4 that will be deployed to a Node that has the label 'owner=john'

1. Label the single node
2. Create the pod

<details><summary>show</summary>
<p>

Add the label to a node:

```bash
# 1
kubectl label nodes <your-node-name> owner=john
kubectl get nodes -L owner
```

```bash
# 2
# Create yml
kubectl run nginx4 --image=nginx $do > nginx4.yml
```
Edit file, use the 'nodeSelector' property on the Pod YAML:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx4
  name: nginx4
spec:
  containers:
    - image: nginx
      name: nginx4
      resources: {}
  nodeSelector:         #ADDED
    owner: john         #ADDED
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
  ~

```

You can easily find out where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

OR:
Use node affinity (https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-required-node-affinity)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
    ...
```

</p>
</details>

### L8 Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

#or

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### L8 Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
kubectl annotate pod nginx1 --list
  
# or

kubectl describe po nginx1 | grep -i 'annotations'

# or

kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

As an alternative to using `| grep` you can use jsonPath like `kubectl get po nginx1 -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### L8 Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>


## <a name="Deployments">Deployments</a>

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### D1 Common Deploy operations
In namespace d1
1. Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)
2. View the YAML of this deployment
3. Check how the deployment rollout is going
4. View the YAML of the replica set that was created by this deployment
5. Update the nginx image to nginx:1.19.8
6. Check the rollout history and confirm that the replicas are OK
7. Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)
8. Do an on purpose update of the deployment with a wrong image nginx:1.91. Verify that something's wrong with the rollout
9. Find from the history the rollout with version 1.18.0 and restore it.
10. Scale the deployment to 5 replicas
<details><summary>show</summary>
<p>

```bash
# -- 0
kubectl -n d1 create ns d1
#-- 1
kubectl -n d1 create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80

#-- 2
kubectl -n d1 get deploy nginx -o yaml


#--3
kubectl -n d1 describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl -n d1 rollout status deploy nginx

#-- 4
kubectl -n d1 describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR
kubectl -n d1 get rs -l app=nginx # if you created deployment by 'create' command

kubectl -n d1 rollout status deploy nginx
# you could also just do kubectl get rs
kubectl -n d1 get rs nginx-7bf7478b77 -o yaml


#-- 5
kubectl -n d1 set image deploy nginx nginx=nginx:1.19.8
# alternatively...
kubectl -n d1 edit deploy nginx # change the .spec.template.spec.containers[0].image

#-- 6
kubectl -n d1 rollout history deploy nginx
kubectl -n d1 get deploy nginx
kubectl -n d1 get rs # check that a new replica set has been created
kubectl -n d1 get po

#-- 7
kubectl -n d1 rollout history deployment.apps/nginx --revision 3
# Indicates the correct image 
kubectl -n d1 rollout undo deploy nginx
# wait a bit
kubectl -n d1 get po # select one 'Running' Pod
kubectl -n d1 describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.18.0


#-- 8
kubectl -n d1 set image deploy nginx nginx=nginx:1.91

kubectl -n d1 rollout status deploy nginx
# or
kubectl -n d1 get po # you'll see 'ErrImagePull' or 'ImagePullBackOff'


#-- 9
# Check the revision with the desired image
kubectl rollout history deployment.apps/nginx --revision 3
kubectl -n d1 rollout undo deploy nginx --to-revision=3

kubectl -n d1 describe deploy nginx | grep Image:
kubectl -n d1 rollout status deploy nginx # Everything should be OK

#-- 10
kubectl -n d1 scale deploy nginx --replicas=5
kubectl -n d1 get po
kubectl -n d1 describe deploy nginx
```

</p>
</details>

### D2 Rollout strategy
In namespace d2
1. Create a deployment named nginx using image nginx 1.18.0  with 8 replicas and ensure that no less that 2 replicas are alive during rollout
2. Set image of the deployment to 1.19.8 and ensure smooth transition
<details><summary>show</summary>
<p>

```bash
# 0 
kubectl create ns d2

# 1.1 
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=8 $do > d2.yaml
 ```
-  Edit Yaml file 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 8
  selector:
    matchLabels:
      app: nginx
  strategy: # The strategy section has been updated
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.18.0
        name: nginx
        resources: {}
status: {}

```
```bash

# 1.2  
kubectl -n d2 apply -f d2.yaml

# 2
kubectl -n d2 set image deployment.apps/nginx nginx=nginx:1.19.8
# Watch the pod progress 
 ```

</p>
</details>

### D3 Pause - Resume deployments
In namespace d3
1. Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes 
2. Pause the rollout for the deploy
3. Update the nginx image to nginx:1.19.8
4. Verify that nothing happens
5. Resume the deployment
6. Verify that image 1.19.8 is in use

<details><summary>show</summary>
<p>

```bash
# -- 0
kubectl create ns d3
#-- 1
kubectl -n d3 create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80

#-- 2
kubectl -n d3 rollout pause deployment nginx
#-- 3
kubectl -n d3 set image deploy nginx nginx=nginx:1.19.8
#-- 4
kga -n d3 # no new replica set
#-- 5
kubectl -n d3 rollout resume deploy nginx
#-- 6
kga -n d3 #  new replica set, deploy indicates expected image version
```

</p>
</details>

### D16 Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
# view the horizontalpodautoscalers.autoscaling for nginx
kubectl get hpa nginx
```

</p>
</details>



### D20 Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary><p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p></details>

## <a name="Helm">Helm Management</a>


### H0 List the basic entities of Helm

<details><summary>show</summary>

<p>
- Chart
- Repository 
- Release

</p></details>



### H1 Repository basic operations
1. Add repository kubernetes-dashboard with URL https://kubernetes.github.io/dashboard/
2. List repositories
3. Update repository
4. Delete repository kubernetes-dashboard
5. Search repository for artifact nginx
6. List everything in bitnami repository
7. Search repository for artifact nginx list all versions

<details><summary>show</summary><p>

```bash
# 1 
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# 2
helm repo list
# 3
helm repo update
# 4
helm repo remove kubernetes-dashboard
# 5
helm search repo nginx
# 6
helm search repo bitnami
```
</p></details>

### H2 Release Helm basic operations
In namespace hel2
1. Deploy chart nginx with generated name
2. List currently installed charts
3. Delete  installed release in (1)
4. Deploy chart nginx, named nginx-release-latest 
5. Deploy chart nginx, version 11.1.5, named nginx-release-11-1-5
6. Upgrade release nginx-release-11-1-5 to the latest nginx chart
<details><summary>show</summary><p>

```bash
# 1 
kubectl create ns hel2
helm repo update
helm -n hel2 install bitnami/nginx --generate-name 
# 2
helm-n hel2 list 
# OR 
helm -n hel2 ls -a 
# 3
helm -n hel2 uninstall nginx-1654263935
# 4
helm -n hel2 install nginx-release-latest bitnami/nginx
helm -n hel2 list
# 5
helm -n hel2 install nginx-release-11-1-5 bitnami/nginx  --version 11.1.5
# 6
helm -n hel2 upgrade nginx-release-11-1-5 bitnami/nginx
```
</p></details>


### H3 Helm charts parameter use
In namespace hel3
1. Check the parameters of chart bitnami/apache, find the parameter for number of replicas
2. install chart using a number of replicas set to 3 vi Helm values

<details><summary>show</summary><p>

```bash
kubectl create namespace hel3
helm repo update
# 1 
helm search repo bitnami/apache
helm show values bitnami/apache |  grep replica -C10
# Value to be set is replicaCount
# 2
helm -n hel3 install my-apache bitnami/apache --set replicaCount=3
# Verify
curl <CLUSTER-IP>
kubectl delete ns hel3
```
</p></details>


## <a name="bluegreen">Canary Blue/Green Deployments</a>


### BG.1  Blue green deploy
A blue deployment was created and running as  follows:
```bash
# Existing blue deploy
kubectl create ns bg
kubectl -n bg create deploy nginx-v14 --image=nginx:1.14 --replicas=2 --port=80
kubectl -n bg expose deploy nginx-v14 --name nginx-svc --port=80
```
1. Create a green  deploy with nginx version 1.22 named nginx-v22 as the original deploy
2. Create temporary svc  to test green deploy
3. Promote green as blue
4. Delete previous blue deployment and interim green service


<details><summary>show</summary><p>

```bash
# 1 
kubectl -n bg create deploy nginx-v22 --image=nginx:1.22 --replicas=2 --port=80
# 2
kubectl -n bg expose deployment.apps/nginx-v22 --name nginx-green --port=80
# 3
kubectl -n bg delete svc  nginx-svc; \
kubectl -n bg expose deployment.apps/nginx-v22 --name nginx-svc --port=80
# 4
kubectl -n bg delete deployment.apps/nginx-v14
kubectl -n bg delete svc nginx-green

```
</p></details>

## C1 Canary Deployment
An application is created and running as following

<details><summary>show</summary>
<p>
See in service exercise "SVC.5 Deployment/Service Expose service based on selector"
```
</p>
</details>


## <a name="record">Record</a>
### TODO 
- green/blue deploys
### Missed
- L7, L8