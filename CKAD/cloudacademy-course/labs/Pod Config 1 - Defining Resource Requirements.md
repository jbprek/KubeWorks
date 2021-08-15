````bash
# 1. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

#2. Create a Pod manifest for a Pod that will consume a lot of CPU resources:

cat << 'EOF' > load.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load
spec:
  containers:
  - name: cpu-load
    image: cloudacademydevops/stress
    args:
    - -cpus
    - "2"
EOF
#The stress image runs a binary that can consume a varying amount of resources. 
# The args instruct stress to attempt to consume two CPU cores, which is how many cores each node in the cluster has.

#3. Create the Pod:
kubectl create -f load.yaml

#4. Display the help document for the top command:
kubectl top --help
#The top command is similar to the native Linux top command for measuring CPU and memory usage of processes. 
# You can monitor at the node or pod level.

 

#5. List the resource consumption of pods:
kubectl top pods

#6. Confirm that the Pod is using almost all the CPU for one of the nodes in the cluster:
kubectl top 

#kubectl top nodes
cat << 'EOF' > load-limited.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-limited
spec:
  containers:
  - name: cpu-load-limited
    image: cloudacademydevops/stress
    args:
    - -cpus
    - "2"
    resources:
      limits:
        cpu: "0.5" # half a core
        memory: "20Mi" # 20 mebibytes 
      requests:
        cpu: "0.25" # quarter of a core
        memory: "10Mi" # 20 mebibytes
EOF

#The resources key is added to specify the limits and requests. 
# The Pod will only be scheduled on a Node with 0.25 CPU cores and 10MiB of memory available. 
# It's important to note that the scheduler doesn't consider the actual resource utilization of the node. 
# Rather, it bases its decision upon the sum of container resource requests on the node. 
# For example, if a container requests all the CPU of a node but is actually 0% CPU, the scheduler would treat the node as not having any CPU available.

#  8. Create the Pod with the resource constraints:
kubectl create -f load-limited.yaml

# 9. Wait a minute for the new Pod's metrics to be collected, and then display the Pod resource utilization:
 kubectl top pods
 
# 10. Delete the Pods created in this lab step:
kubectl delete -f load.yaml
kubectl delete -f load-limited.yaml

````