Check 1: Persisting Data
Create a PersistentVolume named pv in the qq3 Namespace. The PersistentVolume must be configured with the following settings:

storageClassName: host
2Gi of storage capacity
Allow a single Node read-write access
Use a hostPath of /mnt/data
The PersistentVolume must be claimed by a PersistentVolumeClaim named pvc. The PersistentVolume must request 1Gi of storage capacity.

Lastly, create a Pod named persist with one redis container. The Pod must use the pvc PersistentVolumeClaim to mount a volume at /data.

Kubernetes
Check 2: Volume Mounts
The following manifest declares a single Pod named logger in the blah namespace. The logger Pod contains two containers c1 and c2. Before applying this manifest into the cluster, update it so that it declares a hostPath based Volume named vol1 with the host path set to /tmp/vol, and then mount this volume into both the c1 and c2 containers using the container directory /var/log/blah.

apiVersion: v1
kind: Pod
metadata:
labels:
env: prod
name: logger
namespace: blah
spec:
containers:
- image: bash
  name: c1
  command: ["/usr/local/bin/bash", "-c"]
  args:
  - ifconfig > /var/log/blah/data;
    sleep 3600;
- image: bash
  name: c2
  command: ["/usr/local/bin/bash", "-c"]
  args:
  - sleep 3600;

Kubernetes

QUestion : hostPath in volume