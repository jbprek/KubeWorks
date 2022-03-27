# Mysql stateful deployment

##  Kind Configuration
````yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30000
        hostPort: 30000
        protocol: TCP
      - containerPort: 30001
        hostPort: 30001
        protocol: TCP
      - containerPort: 30002
        hostPort: 30002
        protocol: TCP
      - containerPort: 30003
        hostPort: 30003
        protocol: TCP
      - containerPort: 30004
        hostPort: 30004
        protocol: TCP
    extraMounts:
      - hostPath: /home/john/my-kind/vol0
        containerPath: /vol0
      - hostPath: /home/john/my-kind/vol1
        containerPath: /vol1
      - hostPath: /home/john/my-kind/vol2
        containerPath: /vol2
      - hostPath: /home/john/my-kind/vol3
        containerPath: /vol3
      - hostPath: /home/john/my-kind/vol4
        containerPath: /vol4
````
## Create PV, PVC
````bash
cat << EOF > mysql-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/vol3"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
EOF

kubectl apply -f mysql-pv.yaml
````

## Create MySQL service
````bash
cat << EOF > mysql-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    targetPort: 3306
    protocol: TCP
    nodePort: 30000
  selector:
    app: mysql
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:latest
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: dada
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pvc

EOF

kubectl apply -f mysql-svc.yaml
````
#Cleanup
````bash
kubectl delete svc mysql  
kubectl delete deploy mysql
kubectl delete pvc mysql-pvc
kubectl delete pv mysql-pv
````