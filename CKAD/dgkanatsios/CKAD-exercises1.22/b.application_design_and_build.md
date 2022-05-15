#Application  Design  and  Build 20%

## Topics
- [Define, build and modify container images](#docker-podman) 
- Understand [Jobs](#jobs) and [CronJobs](#cronjobs)   
- [Understand  multi-container  Pod  design patterns](#mcp)
- [Utilize  persistent  and  ephemeral  volumes](#per)

## CKAD exam ex

Question 11 | Working with Containers Task weight: 7%

Question 3 | Job Task weight: 2%
Question 16 | Logging sidecar Task weight: 6%
Question 17 | InitContainer Task weight: 4%
Question 22 | Labels Annotations: 3%

Question 12 | Storage, PV, PVC, Pod volume Task weight: 8%
Question 13 | Storage, StorageClass, PVC  Task weight: 6%


## <a name="docker-podman">Define, build and modify container images</a>
**Define, build and modify container images**
### Run httpd detached container, mount volume with html content
- Before : Run the following commands
````bash
# Setup a webpage
mkdir -p ~/dummy/webfiles
echo "Hello From Docker!" > ~/dummy/webfiles/index.html
 ````

1. Create a container in detached mode
   1. Using image httpd:latest
   2. Expose port 80 on 8080
   3. Name it my-apache
   4. mount /dummy/webfiles to /usr/local/apache2/htdocs/
2. Verify that the container is running properly by accessing home page
3. Attach to the container inspect the filesystem
4. Cleanup

<details><summary>show</summary>
<p>

````bash
# 1. Run the container
docker run -d --name=my-apache -p 8080:80 -v /home/john/dummy/webfiles:/usr/local/apache2/htdocs/ httpd:latest
#2. Verify that the container is running properly by accessing home page
curl localhost:8080
#3. Attach to the container inspect the filesystem
docker exec -it my-apache /bin/sh
# 4. Cleanup
docker stop my-apache
docker rm my-apache
rm -rf ~/dummy/webfiles
 ````

</p>
</details>

### Create container connect interactively (GVS)
1. Run the latest version of Ubuntu in container in interactive mode starting a bash shell and explore the/etc/os-release file as well the kernel version uname -r
2. Disconnect from the container without shutting it down (Detach)
3. Reconnect to the shell created in (1)
4. Create a new shell connecting to the container using exec, exit from the shell
5. Terminate container using command in shell(3)
6. Cleanup
<details><summary>show</summary>
<p>

Remember **Ctrl+P + Ctrl+Q** sequence
````bash
# 1. Run the container
docker run -it --name=ubuntu ubuntu
# wait prompt to appear then you may issue any command
# 2. Press Ctrl+P + Ctrl+Q
# 3. Reconnect to the shell created in (1)
docker attach ubuntu
# wait prompt to appear
# 4. Create a new shell connecting to the container using exec, exit from the shell
# In Another Term
docker exec -it ubuntu bash
# run ps or who and we can see 2 sessions
# run exit or Ctlr-D on the prompt
# 5. Terminate container using command in shell(3)
# Switch to term (3) run exit or Ctlr-D on the prompt
# 6. Cleanup
docker rm ubuntu
 ````
</p>
</details>

### Container inspection
 Run a container in detached mode using image nginx find it's IP address

<details><summary>show</summary>
<p>

````bash
# Run container
docker run -d nginx
# Find container id
docker ps
#Output
# CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
# b31c87385e0a   nginx     "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   80/tcp    competent_gauss

docker inspect b31c87385e0a --format=="{{.NetworkSettings.IPAddress}}"
#  Easier
docker inspect b31c87385e0a | grep IPAddress
````
</p>
</details>

### Run Container using environment variable check logs

Issue the following command and inspect and fix the problem

docker run --name=mydb  -d mariadb

<details><summary>show</summary>
<p>

````bash
 docker run --name=mydb  -d mariadb
 # Docker ps shows that the container is not running
 docker ps
 # Inspect the logs
 docker logs mydb
 # Output 
  docker logs mydb
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.7.3+maria~focal started.
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.7.3+maria~focal started.
2022-05-07 12:47:12+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD
        
# Remove the existing container
docker rm mydb 
# Issue the right command
docker run --name=mydb  -d -e MARIADB_ROOT_PASSWORD=changeme mariadb     
````
</p>
</details>

###  Container Image (nginx with custom web content)
- Before : Run the following commands
````bash
# Setup a webpage
mkdir -p ~/dummy/nginxproject/webfiles
echo "Hello From Docker!" > ~/dummy/nginxproject/webfiles/index.html
 ````


<details><summary>show</summary>
<p>

````bash
 docker run --name=mydb  -d mariadb
 # Docker ps shows that the container is not running
 docker ps
 # Inspect the logs
 docker logs mydb
 # Output 
  docker logs mydb
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.7.3+maria~focal started.
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-05-07 12:47:12+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.7.3+maria~focal started.
2022-05-07 12:47:12+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD
        
# Remove the existing container
docker rm mydb 
# Issue the right command
docker run --name=mydb  -d -e MARIADB_ROOT_PASSWORD=changeme mariadb     
````
</p>
</details>

## <a name="jobs">Jobs</a>

### Job creation, monitor, logging
A. Create a job 
1. named pi with 
2. image perl that 
3. runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

B. 
1. Wait till it's done 
2. get the output

<details><summary>show</summary>
<p>


```bash
# A. 
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```
OR

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl logs job/pi
kubectl delete job pi
```
OR

```bash
kubectl wait --for=condition=complete --timeout=300 job pi
kubectl logs job/pi
kubectl delete job pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```

Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>

## <a name="cronjobs">Cron jobs</a>

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.startingDeadlineSeconds=17

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.jobTemplate.spec.activeDeadlineSeconds=12

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 12 # add this line
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>


## <a name="mcp">Multi-container Pods</a>

Exercises in ns mcp dir ~/mcp

### 1. Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# or you can do the above with just an one-liner
kubectl exec -it busybox -c busybox2 -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### 2. Init Container 
Create pod with :
- nginx container exposed at port 80. 
- Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". 
- Make a volume of type emptyDir and mount it in both containers. 
- For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run web --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
  - image: nginx
...
    volumeMounts:
    - name: vol
      mountPath: /usr/share/nginx/html
  volumes:
  - name: vol
    emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - wget -O /work-dir/index.html http://neverssl.com/online
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

In total you get:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - wget -O /work-dir/index.html http://neverssl.com/online 
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- IP"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>


### 3. CKAD Question 16 | Logging sidecar 6%

In namespace mcp
. There is an existing container named cleaner-con in Deployment cleaner in Namespace mcp. This container mounts a volume and writes logs into a file called cleaner.log.

The yaml for the existing Deployment is available at ~/mcp/cleaner.yaml. Persist your changes at ~/mcp/cleaner-new.yaml but also make sure the Deployment is running.

Create a sidecar container named logger-con, image busybox:1.31.0 , which mounts the same volume and writes the content of cleaner.log to stdout, you can use the tail -f command for this. This way it can be picked up by kubectl logs.

Check if the logs of the new container reveal something about the missing data incidents.

#### Setup
```bash

cat << EOF > ~/mcp//cleaner.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  name: cleaner
spec:
  replicas: 2
  selector:
    matchLabels:
      id: cleaner
  template:
    metadata:
      labels:
        id: cleaner
    spec:
      volumes:
      - name: logs
        emptyDir: {}
      initContainers:
      - name: init
        image: bash:5.0.11
        command: ['bash', '-c', 'echo init > /var/log/cleaner/cleaner.log']
        volumeMounts:
        - name: logs
          mountPath: /var/log/cleaner
      containers:
      - name: cleaner-con
        image: bash:5.0.11
        args: ['bash', '-c', 'while true; do echo `date`: "remove random file" >> /var/log/cleaner/cleaner.log; sleep 1; done']
        volumeMounts:
        - name: logs
          mountPath: /var/log/cleaner
EOF

kubectl apply -f ~/mcp/cleaner.yml
```

#### Cleanup
```bash
kubectl delete ns mcp
```

<details><summary>show</summary>
<p>



```bash
cp dummy/cleanup.yml dummy/cleanup-new.yml
```

Edit cleanup-new.yml as follows 
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
   creationTimestamp: null
   name: cleaner
   namespace: mercury
spec:
   replicas: 2
   selector:
      matchLabels:
         id: cleaner
   template:
      metadata:
         labels:
            id: cleaner
      spec:
         volumes:
            - name: logs
              emptyDir: {}
         initContainers:
            - name: init
              image: bash:5.0.11
              command: ['bash', '-c', 'echo init > /var/log/cleaner/cleaner.log']
              volumeMounts:
                 - name: logs
                   mountPath: /var/log/cleaner
         containers:
            - name: cleaner-con
              image: bash:5.0.11
              args: ['bash', '-c', 'while true; do echo Wed May 11 07:40:46 AM UTC 2022: "remove random file" >> /var/log/cleaner/cleaner.log; sleep 1; done']
              volumeMounts:
                 - name: logs
                   mountPath: /var/log/cleaner
# New section below                  
            - name: logger-con
              image: busybox:1.31.0
              args: ['/bin/sh', '-c', 'tail -f /var/log/cleaner/cleaner.log']
              volumeMounts:
                 - name: logs
                   mountPath: /var/log/cleaner

```

```bash
# Apply changes
kubectl apply -f cleanup-new.yml
# Check deployment progress

kubectl -n mercury rollout history deploy cleaner
kubectl -n mercury rollout history deploy cleaner --revision 1
kubectl -n mercury rollout history deploy cleaner --revision 2
# See the logs of a pod's logger container
kubectl -n mercury logs cleaner-799bf8b767-24r6z -c logger-con
```


</p>
</details>

## <a name="per">Utilize  persistent  and  ephemeral  volumes</a>
kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

All exercises on namespace pers

### PersistentVolume
Create a PersistentVolume:
1. 1Gi
2. name 'myvolume'. 
3. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', 
4. storageClassName 'normal', 
5. mounted on hostPath '/etc/foo'. 
6. Save it on pv.yaml, add it to the cluster. 
7. Show the PersistentVolumes that exist on the cluster

<details><summary>show</summary>
<p>

```bash
vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Show the PersistentVolumes:

```bash
kubectl create -f pv.yaml
# will have status 'Available'
kubectl get pv
```

</p>
</details>

### PersistentVolumeClaim

Create a PersistentVolumeClaim for this storage class:
1. called 'mypvc', 
2. a request of 200Mi and an 
3. accessMode of ReadWriteOnce
4. storageClassName of normal, and 
5. save it on pvc.yaml. 
6. Create it on the cluster. 
7. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi
```

Create it on the cluster:

```bash
kubectl create -f pvc.yaml
```

Show the PersistentVolumeClaims and PersistentVolumes:

```bash
kubectl get pvc # will show as 'Bound'
kubectl get pv # will show as 'Bound' as well
```

</p>
</details>

### Pod Using PVC
Create a busybox pod 
1. with command 'sleep 3600', 
2. save it on pod.yaml. 
3. Mount the PersistentVolumeClaim to '/etc/foo'. 
4. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show</summary>
<p>

Create a skeleton pod:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```

Add the lines that finish with a comment:

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
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Create the pod:

```bash
kubectl create -f pod.yaml
```

Connect to the pod and copy '/etc/passwd' to '/etc/foo/passwd':

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### Storage, StorageClass, PVC CKAD 6%
Team Moonpie, which has the Namespace moon, needs more storage.

A. Create a new Storage class using:
1. Name my-sc
2. Provisioner to-be-implemented
3. Retain policy: Retain

B. Create a new PersistentVolumeClaim 
1. named moon-pvc
2. in that namespace moon
4. use w StorageClass my-sc 
5. The claim should request storage of 1Mi,
6.  accessMode of ReadWriteOnce

C. The provisioner to-be-implemented will be created by another team, so it's expected that the PVC will not boot yet. Confirm this by writing the log message from the PVC into a file.




<details><summary>show</summary>
<p>

Create the second pod, called busybox2:

```bash
#A
cat << EOF > my-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-sc
provisioner: to-be-implemented
reclaimPolicy: Retain
EOF

kubectl apply -f  my-sc.yaml

# B
cat << EOF > moon-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: moon-pvc
  namespace: moon
spec:
  storageClassName: my-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
EOF

kubectl apply -f  moon-pvc.yaml

#C
 kubectl -n moon describe pvc moon-pvc
```


 Output
```
Events:
Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
Normal  ExternalProvisioning  5s (x5 over 40s)  persistentvolume-controller  waiting for a volume to be created, either by external provisioner "to-be-implemented" or manually created by system administrator
If the file doesn't show on the second pod but it shows on the first, it has most likely been scheduled on a different node.
```
Copy string "waiting for a volume to be created, either by external provisioner "to-be-implemented" or manually created by system administrator
If the file doesn't show on the second pod but it shows on the first, it has most likely been scheduled on a different node." Into a file


</p>
</details>


## TODO Ephemeral volumes

