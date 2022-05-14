![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (0)


kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

## Namespaces

### CKAD 1% List all namespaces - store then in a file
<details><summary>show</summary>
<p>

```bash
kubectl get ns | cut -d " " -f1 > allns.txt
```
</p>
</details>

### Create a namespace called 'dummy' 

<details><summary>show</summary>
<p>
```bash
kubectl create ns dummy
```
</p>
</details>

### Set 'dummy' as working namespace in context, verify that is the one in use, then set back to default


<details><summary>show</summary>
<p>

```bash
kubectl config set-context --current --namespace=dummy
kubectl config view | grep namespace
kubectl config set-context --current --namespace=default
```
</p>
</details>


### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```
Alternatively

```bash
kubectl get po -A
```
</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

</p>
</details>

## Pods 
### CKAD 2% Pods Namespaces
1. Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container.

2. Your manager would like to run a command manually on occasion to output the status of that exact Pod. Please write a command that does this into ~/dummy/pod1-status-command.sh. The command should use kubectl.

<details><summary>show</summary>
<p>

```bash
# 1.1
kubectl run pod1 --image=httpd:2.4.41-alpine -n default $do > pod1.yml
# 1.2  edit containers[0].name to pod1-container
vi pod1.yml 
# 2
cat << EOF > ~/dummy/pod1-status-command.sh
kubect -n default get po -o jsonpath="{.status.phase}"
EOF 

```
</p>
</details>

### CKAD 4% Pods Namespaces
The board of Team Neptune decided to take over control of one e-commerce webserver from Team Saturn. The administrator who once setup this webserver is not part of the organisation any longer. All information you could get was that the e-commerce system is called my-happy-shop.

Search for the correct Pod in Namespace saturn and move it to Namespace neptune. It doesn't matter if you shut it down and spin it up again, it probably hasn't any customers anyways.

#### Setup
```bash
kubectl create namespace saturn
kubectl -n saturn run web1 --image=nginx 
kubectl -n saturn run web2 --image=nginx 
kubectl -n saturn run web3 --image=nginx 
kubectl -n saturn label pods web2 system=my-happy-shop

kubectl create namespace neptune

```

<details><summary>show</summary>
<p>


```bash
#1 Find in saturn the specific pod
kubectl -n saturn describe pods | grep my-happy-shop -C20
# must be web2
# get yaml
kubectl -n saturn get po web2 -o yaml > web2.yml
# Edit file remove all sections beyond metadata and spec
# Start pod
kubectl -n neptune apply -f web2.yml
# Check if it is running
kubectl -n neptune get po web2
#Delete pod from saturn
kubectl -n saturn delete po web2

```

</p>
</details>

### Create a namespace called 'ns1' and a pod with image nginx called nginx on this namespace

<details><summary>show</summary>
<p>

```bash
kubectl create namespace ns1
kubectl run nginx --image=nginx --restart=Never -n ns1
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n ns1 -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: ns1
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
```

Alternatively, you can run in one line

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n ns1 -f -
# OR (namespace not needed since it's included in yaml)
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl apply -f -

```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --command --restart=Never -it -- env # -it will help in seeing the output
# or to have the pod removed automaticaly use the --rm option
kubectl run busybox --image busybox --restart=Never --rm -it -- env
# or, just run it without -it
kubectl run busybox --image=busybox --command --restart=Never -- env
# and then, check its logs
kubectl logs busybox

#EXAM Prefered using env var
kubectl run busybox --image=busybox $rm -- env
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>


### Create a pod with image nginx called nginx and expose traffic on port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

</p>
</details>

### Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # you will see an event 'Container nginx definition changed, will be restarted'
kubectl get po nginx -w # watch it
```

*Note*: some time after changing the image, you should see that the value in the `RESTARTS` column has been increased by 1, because the container has been restarted, as stated in the events shown at the bottom of the `kubectl describe pod` command:

```
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
[...]
  Normal  Killing    100s                 kubelet, node3     Container pod1 definition changed, will be restarted
  Normal  Pulling    100s                 kubelet, node3     Pulling image "nginx:1.7.1"
  Normal  Pulled     41s                  kubelet, node3     Successfully pulled image "nginx:1.7.1"
  Normal  Created    36s (x2 over 9m43s)  kubelet, node3     Created container pod1
  Normal  Started    36s (x2 over 9m43s)  kubelet, node3     Started container pod1
```

*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>

### Create an long running ubuntu instance, to be used as debug pod, connect using bahs in the instance

<details><summary>show</summary>
<p>

```bash
kubectl run ubuntu --image=ubuntu --command -- /bin/sleep 365d
# then
kubectl exec -it ubuntu -- /bin/bash
```

</p>
</details>


### Copy a file from a container to the localfilesystem
1. create a busybox pod running sleep 3600
2. copy /etc/passwd file from the pod to the local filesystem
3. copy the file from (2) into the pod's /tmp directory

<details><summary>show</summary>
<p>

```bash
#1 
kubectl run bus3 --image=busybox --command -- sh -c "sleep 3600"
# 2 Note missing leading '/'
kubectl cp bus3:etc/passwd ./passwd.txt
#3 Note missing leading '/'
kubectl cp passwd.txt bus3:tmp/
```

</p>
</details>
