````bash

# 1 Create namespace
kubectl create namespace jobs

# Set namespace as the default for the current context
kubectl config set-context --current --namespace=jobs

kubectl config view --minify | grep namespace:

#2. Create a Job named one-off that sleeps for 30 seconds:
kubectl create job one-off --image=alpine -- sleep 30

#4. Use explain to see what other Job spec fields can be specified:

Copy code
1
kubectl explain job.spec | more

#5. Create a Job that has a Pod that always fails:

cat << 'EOF' > pod-fail.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pod-fail
spec:
  backoffLimit: 3
  completions: 6
  parallelism: 2
  template:
    spec:
      containers:
      - image: alpine
        name: fail
        command: ['sleep 20 && exit 1']
      restartPolicy: Never
EOF
kubectl create -f pod-fail.yaml

#The Pod will always fail after sleeping for 20 seconds due to the exit 1 command (returning a non-zero exit code is treated as a failure). The Job allows for two Pods to run in parallel.

#6. Watch the describe output for the Job to see how the Job progresses:

watch kubectl describe jobs pod-fail

#8. Get the Pods in the jobs namespace:
kubectl get pods
## All of the Pods associated with the Jobs you ran are listed. The Pods will remain until you delete them or the Job associated with them. 
# Setting a Job's ttlSecondsAfterFinished can free you from manually cleaning up the Pods.

# 9. Create a CronJob that runs a Job every minute:
cat << 'EOF' > cronjob-example.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-example
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - image: alpine
            name: fail
            command: ['date']
          restartPolicy: Never
EOF
kubectl create -f cronjob-example.yaml

#10. Observe the Events of the CronJob to confirm that it is creating Jobs every minute: 
watch kubectl describe cronjob cronjob-example
````