# Mysql stateful deployment

## Create PV, PVC
````bash
cat << EOF > mysql-minikube.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv  
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
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
      storage: 1Gi

  
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

---
apiVersion: v1
kind: Service
metadata:
  name: mysql  
spec:
  type: ClusterIP
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
EOF

kubectl apply -f mysql-minikube.yaml
````
# User port forward to access the service from client tools
````bash
kubectl port-forward service/mysql 3306:3306
````
#Cleanup
````bash
kubectl delete svc mysql  -n mysql
kubectl delete deploy mysql -n mysql
kubectl delete pvc mysql-pvc  -n mysql
kubectl delete pv mysql-pv -n mysql
````