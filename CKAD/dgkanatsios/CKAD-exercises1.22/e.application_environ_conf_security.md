# Application  Environment, Configuration  and  Security 25%

- Discover and use resources that extend Kubernetes (CRD)
- Understand authentication, authorization and admission control
- [Understanding and defining resource requirements, limits and quotas](#quotas)
- [Understand ConfigMaps](#cm)
- Create & consume Secrets
- Understand ServiceAccounts
- [Understand SecurityContexts](#secctx)

## TODO Discover and use resources that extend Kubernetes  (CRD)
## TODO Understand  authentication,  authorization and  admission  control


## <a name="quotas">Understanding and defining resource requirements, limits and quotas</a>


### QU.Quiz
Explain the concept of **request** and **limit** properties in manifest files
<details><summary>show</summary>
<p>
- request is the scheduled amount of resources to be allocated
- limit is the upper bound of resource use, that can be allocated beyond the request values.
</p>
</details>

### QU.1 Set Pod Memory and CPU quotas
In namespace qu1 create a Deployment named apache, image httpd:2.4-alpine, and 3 replicas. The containers should be named apache-pod. Each container should have a memory request of 20Mi and a memory limit of 50Mi and a CPU requet of 20m and a limit of 40m


<details><summary>show</summary>
<p>

```bash
kubectl create ns qu1

kubectl -n qu1 create deploy apache --image=httpd:2.4-alpine --replicas=3 $do > qu1.yaml
```

```yaml
# Edited qu1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: apache
  name: apache
  namespace: qu1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apache
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: apache
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: apache-pod
        # The below section has been edited
        resources: 
          requests:
            memory: "20Mi"
            cpu: "20m"
          limits:
            memory: "50Mi"
            cpu: "40m"
status: {}

```

```bash
kubectl apply -f qu1.yaml
```

</p>
</details>

## Requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)


### Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

</p>
</details>


### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

Note: Use of `--requests` and `--limits` flags in the imperative `run` command is deprecated as of 1.21 K8s version and will be removed in the future. Instead, use `kubectl set resources` command in combination with `kubectl run --dry-run=client -o yaml ...` as shown below.

Alternative using `set resources` in combination with imperative `run` command:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml > nginx-pod.yml
```

```bash
kubectl create -f nginx-pod.yml
```

</p>
</details>


## <a name="cm">ConfigMaps</a>

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### CM.1 ConfigMap Literal Values
1. In namespace cm create a configmap named conf-from-lit with values foo=lala,foo2=lolo
2. Display its values

<details><summary>show</summary>
<p>

```bash
# 1
kubectlcreate ns cm
kubectl -n cm create cm conf-from-lit --from-literal=foo=lala --from-literal=foo2=lolo

# 2
kubectl get cm conf-from-lit -o yaml
# or
kubectl describe conf-from-lit config1
```

</p>
</details>

### CM.2  ConfigMap from File

1. Create and display a configmap conf-from-file from a file
2. Display its values

- Setup

```bash
echo -e "foo3=lili\nfoo4=lele" > config2.txt
```
<details><summary>show</summary>
<p>

```bash
# 1
kubectl create cm conf-from-file --from-file=config2.txt
# 2
kubectl get cm conf-from-file -o yaml
```

</p>
</details>

### CM.3 Create and display a configmap named cm conf-from-env-file from a .env file

Note: Substitute for --from-literal

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm conf-from-env-file --from-env-file=config.env
kubectl describe cm conf-from-env-file
kubectl get cm conf-from-env-file -o yaml
```

</p>
</details>

### CM.4 Create and display a configmap named conf-from-file-key  from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > conf-from-file-key.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm conf-from-file-key --from-file=special=conf-from-file-key.txt
kubectl describe cm conf-from-file-key
kubectl get cm conf-from-file-key -o yaml
```

</p>
</details>

### CM.5 Create a new nginx pod that loads the value from variable 'var2', from conf-from-env-file  in an env variable called 'VAR2'

<details><summary>show</summary>
<p>

[Solution temlate from docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap) 

```bash
kubectl run nginx1 --image=nginx $do > nginx1.yml
vi nginx1.yaml
```

Edit generated pod.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      env:
        # Define the environment variable
        - name: option
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: config3
              # Specify the key associated with the value
              key: var2
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

```bash
kubectl create -f nginx1.yaml
kubectl exec -t nginx1 -- env | grep VAR2 # will show 'VAR2=val2'
```

</p>
</details>

###  CM.6 Load conf-from-env-file CM as env variables into a new nginx pod named nginx2
<details><summary>show</summary>
<p>

Note effective diff from the previous is that all key-value pairs will be mapped to the env.

```bash
kubectl run  nginx2 --image=nginx $do > nginx2.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx2
  name: nginx2
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: conf-from-env-file # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f nginx2.yaml
kubectl exec -t nginx2 -- env 
```

</p>
</details>

### CM.7 Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

<details><summary>show</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

</p>
</details>



### CM.8 Mount a page in nginx
- Mount an HTML page under a specific URL (/page.html) on a Nginx Pod named nginx3 using CM
- Contents of somepage.html
```bash

cat << EOF > somepage.html
<!doctype html>
<html>
<head>
    <title>Loaded from Config Map</title>
</head>
<body>
<p>This is an example paragraph. Anything in the <strong>body</strong> tag will appear on the page, just like this <strong>p</strong> tag and its contents.</p>
</body>
</html>
EOF
```

Make this page to be available under http://<url>/page.html of an nginx pod


Create a configMap 'web-page-cm' with th e.

<details><summary>show</summary>
<p>

```bash
kubectl create cm page-cm --from-file=page.html=somepage.html
kubectl run nginx3 --image=nginx $do > nginx3.yaml
vi nginx3.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx3
  name: nginx3
spec:

  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: page-cm
  containers:
    - image: nginx
      name: nginx3
      resources: {}
      volumeMounts:
        - name: config-volume
          mountPath: /usr/share/nginx/html
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

</p>
</details>




## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative using `--jsonpath`:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d  # on MAC it is -D
```

Alternative using `--template`:

```bash
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d  # on MAC it is -D
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret2 # our secret's name
          key: username # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```
Alternatively

```bash
kubectl get sa -A
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
kubectl create sa myuser
```

Alternatively:

```bash
# let's get a template easily
kubectl get sa default -o yaml > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### Create an nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run=client > pod.yaml
kubectl apply -f pod.yaml
```

or you can add manually:

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: myuser # we use pod.spec.serviceAccountName
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

or

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccount: myuser # we use pod.spec.serviceAccount
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
kubectl describe pod nginx # will see that a new secret called myuser-token-***** has been mounted
```


</p>
</details>



## <a name="secctx">Understand SecurityContexts</a>




### SCTX.2  Security context, pod level and container level setting

In namespace sctx
Create a pod named scbus 
- image busybox
- with an emptyDir volume mounted at /demo
- with pod running as user 1000 group 2000 filesystem 2000
- container with no privilege escalation

<details><summary>show</summary>
<p>

```bash
kubectl -n sctx run scbus --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c "sleep 1d" > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: scbus
  name: scbus
  namespace: sctx
spec:
  volumes:
    - name: shared-data
      emptyDir: {}

  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - args:
        - /bin/sh
        - -c
        - sleep 1d
      image: busybox
      volumeMounts:
        - name: shared-data
          mountPath: /demo
      name: scbus
      securityContext:
        allowPrivilegeEscalation: false
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

</p>
</details>



### SCTX.2  Security context, container capabilities

In namespace sctx
Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container


<details><summary>show</summary>
<p>

```bash
kubectl -n sctx run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: scbus
  name: scbus
  namespace: sctx
spec:
  volumes:
    - name: shared-data
      emptyDir: {}

  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - args:
        - /bin/sh
        - -c
        - sleep 1d
      image: busybox
      volumeMounts:
        - name: shared-data
          mountPath: /demo
      name: scbus
      securityContext:
        allowPrivilegeEscalation: false
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

</p>
</details>
