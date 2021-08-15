#Introduction
If a Pod is in a Running state it does not necessarily mean the Pod is ready to use or that the Pod is operating as you would expect. For example, a running Pod may not be ready to accept requests or may have entered into an internal failed state, such as a deadlock, preventing it to make progress on new requests. Kubernetes introduces probes for containers to detect when Pods are in such states. More specifically:

 - Readiness probes are used to detect when a Pod is unable to serve traffic, such as during startup when large amounts of data are being loaded or caches are being warmed. When a readiness probe fails, it means that the Pod needs more time to become ready to serve traffic. When the Pod is accessed through a Kubernetes Service, the Service will not serve traffic to any Pods that have a failing readiness probe.
 - Liveness probes are used to detect when a Pod fails to make progress after entering a broken state, such as deadlock. The issues causing the Pod to enter such a broken state are bugs, but by detecting a Pod is in a broken state allows Kubernetes to restart the Pod and allow progress to be made, perhaps only temporarily until arriving at the broken state again. The application availability is improved compared to leaving the Pod in the broken state.
 - Startup probes are used when an application starts slowly and may otherwise be killed due to failed liveness probes. The startup probe runs before both readiness and liveness probes. The startup probe can be configured with a startup time that is longer than the time needed to detect a broken state for a container after it has started.

Readiness and Liveness probes run for the entire lifetime of the container they are declared in. Startup probes only run until they first succeed. A container can define up to one of each type of probe. All probes are also configured the same way. The only difference is how a probe failure is interpreted.

You will use both Readiness and Liveness probes in this lab step.
````bash
# Create namespace

# 1. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

# 2. Explain the readinessProbe field which is at the path pod.spec.containers.readinessProbe:
kubectl explain pod.spec.containers.readinessProbe
````
Read through the DESCRIPTION and FIELDS. There are three types of actions a probe can take to assess the readiness of a Pod's container:

- exec: Issue a command in the container. If the exit code is zero the container is a success, otherwise it is a failed probe.
- httpGet: Send and HTTP GET request to the container at a specified path and port. If the HTTP response status code is a 2xx or 3xx then the container is a success, otherwise, it is a failure.
- tcpSocket: Attempt to open a socket to the container on a specified port. If the connection cannot be established, the probe fails.

The number of consecutive successes is configured via the successThreshold field and the number of consecutive failures required to transition from success to failure is failureThreshold. The probe runs every periodSeconds and each probe will wait up to timeoutSeconds to complete.

````bash
# 3. Create a Pod that uses a readinessProbe with an HTTP GET action:
cat << 'EOF' > pod-readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-http
spec:
  containers:
  - name: readiness
    image: httpd:2.4.38-alpine
    ports:
    - containerPort: 80
    # Sleep for 30 seconds before starting the server
    command: ["/bin/sh","-c"]
    args: ["sleep 30 && httpd-foreground"]
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
kubectl create -f pod-readiness.yaml

#4. Describe the Pod to see how events related to the readiness probe:
kubectl describe pod readiness-http

#5. To confirm the readiness probes are always running, kill the httpd server processes running in the container
kubectl exec readiness-http -- pkill httpd

#6. Describe the Pod and observe the Ready Conditions are False:
kubectl describe pod readiness-http
#7. Create a Pod that uses a liveness probe to detect broken states:
#Recall that liveness probes have the same configuration as readiness probes. 
# The nc (netcat) command listens (the -l option) for connections on port (-p) 8888 and responds with the message hi for the first 30 seconds, after which timeout kills the nc server. 
# The liveness probe attempts to establish a connection with the server every 5 seconds.
 

cat << 'EOF' > pod-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-tcp
spec:
  containers:
  - name: liveness
    image: busybox:1.30.1
    ports:
    - containerPort: 8888
    # Listen on port 8888 for 30 seconds, then sleep
    command: ["/bin/sh", "-c"]
    args: ["timeout 30 nc -p 8888 -lke echo hi && sleep 600"]
    livenessProbe:
      tcpSocket:
        port: 8888
      initialDelaySeconds: 3
      periodSeconds: 5
EOF
kubectl create -f pod-liveness.yaml

#8. Watch the describe output for the Pod to observe when the liveness probe fails and the Pod is restarted:
watch kubectl describe pod liveness-tcp

#9. Press ctrl+c to stop watching the output.

# 10. Delete the Pods created in this lab step:
kubectl delete -f pod-readiness.yaml
kubectl delete -f pod-liveness.yaml
````

#Monitoring Kubernetes Applications
There are many metrics that you may want to keep track of in Kubernetes: available node CPU/Memory, number of running Pods vs. desired Pods for Deployments, errors and warnings, etc. There are several built-in mechanisms available to make monitoring and debugging applications easier. You can also leverage purpose-built monitoring systems that run in Pods. Kubernetes has its own basic monitoring solution called Metrics Server. You will use Metrics Server later in this lab step. You can also use more complete metrics pipelines, such as Prometheus, but that is beyond the scope of this lab. 

````bash
#1. Use the get command to ensure all Pods have all containers running:
kubectl get pods --all-namespaces

#2. List all the events in the logs namespace:
kubectl get events -n logs

# 3. Enter the following command to see how to use the top command to monitor node and Pod resource utilization:
kubectl top
#4. Use top to view resource utilization metrics for the cluster's nodes:
kubectl top node
#5. Use top to view resource utilization metrics for the Pods in the logs Namespace:
kubectl top pods -n logs
#6. Display the resource utilization of individual containers:
kubectl top pod -n logs --containers
#7. Use a label selector to show only resource utilization for Pods with a test label:
kubectl top pod -n logs --containers -l test

````