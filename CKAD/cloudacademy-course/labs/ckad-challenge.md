#Certified Kubernetes Application Developer (CKAD) Challenge

##CKAD Check 1: Service Account
Create a service account named inspector in the dwx7eq namespace. Then create a deployment named calins in the same namespace. Use the image busybox:1.31.1 for the only pod container and pass the arguments sleep and 24h to the container. Set the number of replicas to 1. Lastly, make sure that the deployments' pod is using the inspector service account.

Kubernetes
##CKAD Check 2: Evictions
The mission-critical deployment in the bk0c2d namespace has been getting evicted when the Kubernetes cluster is consuming a lot of memory. Modify the deployment so that it will not be evicted when the cluster is under memory pressure unless there are higher priority pods running in the cluster (Guaranteed Quality of Service). It is known that the container for the deployment's pod requires and will not use more than 200 milliCPU (200mi) and 200 mebibytes (200Mi) of memory.



##CKAD Check 3: Persisting Data
A legacy application runs via a deployment in the zuc0co namespace. The deployment's pod uses a multi-container pod to convert the legacy application's raw metric output into a format that can be consumed by a metric aggregation system. However, the data is currently lost every time the pod is deleted. Modify the deployment to use a persistent volume claim with 2GiB of storage and access mode of ReadWriteOnce so the data is persisted if the pod is deleted.

Kubernetes
##CKAD Check 4: Multi-Container Pattern
Write the name of the multi-container pod design pattern used by the pod in the previous task to a file at /home/ubuntu/mcpod.