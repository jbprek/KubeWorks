Introduction
A security context allows you to set access control for Pods, as well as containers and volumes in Pods, when applicable. Examples of access controls that can be set with security contexts include:

- The user ID and group IDs of the first process running in a container
- The group ID of volumes
- If a container's root file system is read-only
- Security-Enhanced Linux (SELinux) options
- The privileged status of containers, which allows the container to do almost everything root can do on the host if enabled
- Whether or not privilege escalation, where child processes can have more privileges than their parent, is allowed

When a security context field is configured for a Pod and one of the Pod's containers, the container's setting takes precedence. Configuring the security context of Pods and containers can greatly reduce the security risk posed by using third-party images.
````bash
# 0. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

#1. Explain the available Pod-level security context fields:
kubectl explain pod.spec.securityContext | more

#2. Explain the 2. Explain the available container-level security context fields:
kubectl explain pod.spec.containers.securityContext | more

#3. Create the following Pod manifest file:
cat << EOF > pod-no-security-context.yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-test-1
spec:
  containers:
  - image: busybox:1.30.1
    name: busybox
    args:
    - sleep
    - "3600"
EOF
#4. Create the Pod and use exec to list the available devices in the container:
kubectl create -f pod-no-security-context.yaml
kubectl exec security-context-test-1 -- ls /dev

# 5. Delete the previous Pod and create a similar Pod that has a privileged container:
kubectl delete -f pod-no-security-context.yaml

cat > pod-privileged.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: security-context-test-2
spec:
  containers:
  - image: busybox:1.30.1
    name: busybox
    args:
    - sleep
    - "3600"
    securityContext:
      privileged: true
EOF

kubectl create -f pod-privileged.yaml

#6. List the devices available in the container:
kubectl exec security-context-test-2 -- ls /dev
#All of the host devices are available including the host file system disk nvme0n1p1. This could be a major security breach and shows the importance of carefully considering if you should ever use a privileged container.

#8. Create another pod that includes a Pod security context as well as a container security context:
kubectl delete -f pod-privileged.yaml
#The Pod security context enforces that container processes do not run as root (runAsNonRoot) and sets the user ID of the container process to 1000.
# The container securityContext sets the container process' user ID to 2000 and sets the root file system to read-only.
cat << EOF > pod-runas.yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-test-3
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - image: busybox:1.30.1
    name: busybox
    args:
    - sleep
    - "3600"
    securityContext:
      runAsUser: 2000
      readOnlyRootFilesystem: true
EOF

kubectl create -f pod-runas.yaml

#9. Open a shell in the container:
kubectl exec security-context-test-3 -it -- /bin/sh
#Notice that the shell prompt is $ and not # indicating that you are not the root user.

#10. List the running processes in the container:
ps
#The USER ID is 2000 illustrating that it is not root and that the container security context overrides the setting in the Pod security context 
# when both security contexts include the same field. Whenever possible you should not run as root.


 











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