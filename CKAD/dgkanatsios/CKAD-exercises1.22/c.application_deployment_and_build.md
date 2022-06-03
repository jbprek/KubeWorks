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

### D1 Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.18.0/image: nginx:1.18.0\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

or,
```bash
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

</p>
</details>

### D2 View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### D3 View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### D4 Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### D5 Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### D6 Update the nginx image to nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.8
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### D7 Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### D8 Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.18.0
```

</p>
</details>

### D10 Do an on purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and Enter, to navigate quickly
```

</p>
</details>

### D11 Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull' or 'ImagePullBackOff'
```

</p>
</details>

### D12 Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### D13 Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### D15 Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
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

### D17 Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### D18 Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.9
# or
kubectl edit deploy nginx
# change the image to nginx:1.19.9
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### D19 Resume the rollout and check that the nginx:1.19.9 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
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


### Question 4 | Helm Management
Task weight: 5%

Team Mercury asked you to perform some operations using Helm, all in Namespace mercury:

Delete release internal-issue-report-apiv1
Upgrade release internal-issue-report-apiv2 to any newer version of chart bitnami/nginx available
Install a new release internal-issue-report-apache of chart bitnami/apache. The Deployment should have two replicas, set these via Helm-values during install
There seems to be a broken release, stuck in pending-upgrade state. Find it and delete it

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