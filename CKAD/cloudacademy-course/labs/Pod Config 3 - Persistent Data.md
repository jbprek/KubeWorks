Introduction
Persistent Volumes (PVs) are Kubernetes resources that represent storage in the cluster. Unlike regular Pod volumes (volumes of the default emptyDir type) which exist only during the lifetime of the Pod containing volume, PVs do not have a lifetime connected to a Pod. Thus, they can be used by multiple Pods over time, or even at the same time. Different types of storage can be used by PVs including NFS, iSCSI, and cloud-provided storage volumes, such as AWS Elastic Block Store (EBS) volumes. The list of supported PVs is here. Pods claim PV resources through Persistent Volume Claims (PVCs). A Pod can claim a specific amount of PV storage and an access mode, such as read/write by only one node, through a PVC. The PV will be automatically created as long as the cluster supports dynamic provisioning. This allows you to consider PVCs as storage.

The cluster you are using is running in AWS and is configured to automatically create EBS volumes when using PVCs. You will demonstrate how  in this lab step


````bash
# 0. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

#1. Create a Namespace for the resources you'll create in this lab step and change your default kubectl context to use the Namespace:
# Create namespace
kubectl create namespace persistence
# Set namespace as the default for the current context
kubectl config set-context --current --namespace=persistence
#Verify
kubectl config view --minify | grep namespace:

#2. Create a PVC:
cat << 'EOF' > pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-data
spec:
  # Only one node can mount the volume in Read/Write
  # mode at a time
  accessModes:
  - ReadWriteOnce 
  resources:
    requests:
      storage: 2Gi
EOF
kubectl create -f pvc.yaml

#3. Use get to display the PVC:
kubectl get pvc

#4. Use get to display the underlying PV:
kubectl get pv

#5. Create a Pod that mounts the volume provided by the PVC:
cat << 'EOF' > db.yaml
apiVersion: v1
kind: Pod
metadata:
  name: db 
spec:
  containers:
  - image: mongo:4.0.6
    name: mongodb
    # Mount as volume 
    volumeMounts:
    - name: data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: data
    # Declare the PVC to use for the volume
    persistentVolumeClaim:
      claimName: db-data
EOF
kubectl create -f db.yaml

#6. Run the MongoDB CLI client to insert a document that contains the message "I was here" into a test database and then confirm it was inserted:
kubectl exec db -it -- mongo testdb --quiet --eval \
  'db.messages.insert({"message": "I was here"}); db.messages.findOne().message'
  
#7. Delete the Pod:
kubectl delete -f db.yaml

#8. Create a new database Pod:
kubectl create -f db.yaml

#9. Attempt to find a document in the test database:
kubectl exec db -it -- mongo testdb --quiet --eval 'db.messages.findOne().message'
````