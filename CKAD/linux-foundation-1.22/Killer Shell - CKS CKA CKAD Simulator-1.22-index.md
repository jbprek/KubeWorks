## Question 1 | Namespaces Task weight: 1% - List and store namespaces in a file
## Question 2 | Pods Task weight: 2% - Create pod, log status in file
## Question 3 | Job Task weight: 2% - Create a job
## Question 4 | Helm Management Task weight: 5%
## Question 5 | ServiceAccount, Secret Task weight: 3% - Get a secret that belongs to a Service Account
## Question 6 | ReadinessProbe Task weight: 7% - Create a readiness probe
## Question 7 | Pods, Namespaces  Task weight: 4% - Move Pod to a namespace
## Question 8 | Deployment, Rollouts Task weight: 4% - A deployment does not work, rollback to a previous working deployment
## Question 9 | Pod -> Deployment Task weight: 5% - Convert a pod to deployment
## Question 10 | Service, Logs Task weight: 4% - Create ClusterIP service, store service output and pod logs
## Question 11 | Working with Containers Task weight: 7% - Create run container using podman and docker.
## Question 12 | Storage, PV, PVC, Pod volume Task weight: 8% - Create PV, PVC and deploy using PVC
## Question 13 | Storage, StorageClass, PVC  Task weight: 6% - Create PVC using Storage Class using specific provisioner.
## Question 14 | Secret, Secret-Volume, Secret-Env Task weight: 4% - Create Secret, use existing secret
## Question 15 | ConfigMap, Configmap-Volume Task weight: 5% - Create Config map, use if from a pod
## Question 16 | Logging sidecar Task weight: 6% - Create a sidecar for an existing deploy.
## Question 17 | InitContainer Task weight: 4% - Create an initcontainer for a deploy
## Question 18 | Service misconfiguration Task weight: 4% -
## Question 19 | Service ClusterIP->NodePort Task weight: 3% -
## Question 20 | NetworkPolicy Task weight: 9%
## Question 21 | Requests and Limits, ServiceAccount Task weight: 4%




Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container.

Your manager would like to run a command manually on occasion to output the status of that exact Pod. Please write a command that does this into /opt/course/2/pod1-status-command.sh. The command should use kubectl.


