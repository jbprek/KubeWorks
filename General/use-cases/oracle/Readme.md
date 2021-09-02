# Mysql stateful deployment

##  Kind Configuration

See example configuration : Local Registry, 5 Nodeports, 5 Volumes

## Create PV, PVC
````bash
cat << EOF > oracle12-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ora12-pv
  labels:
    type: local
spec:
  storageClassName: ""
  claimRef: 
    name: ora12-pvc
    namespace: dev
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/vol0"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ora12-pvc
  namespace: dev
spec:
  storageClassName: ""
  volumeName: ora12-pv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
EOF

kubectl apply -f oracle12-pv.yaml
````

## Create Oracle 12 service
### Push image to local registry
````bash
docker tag store/oracle/database-enterprise:12.2.0.1 localhost:5000/database-enterprise:12.2.0.1
docker push  localhost:5000/database-enterprise:12.2.0.1
````

### PV, PVC, SVC
````bash
cat << EOF > ora12-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: ora12-svc
spec:
  type: NodePort
  ports:
  - port: 1521
    targetPort: 1521
    protocol: TCP
    nodePort: 30000
    name: ora12-ls-port
  - port: 5500
    targetPort: 5500
    protocol: TCP
    nodePort: 30001
    name: ora12-em-port
  selector:
    app: ora12-ngin
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ora12-deploy
spec:
  selector:
    matchLabels:
      app: ora12-ngin
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: ora12-ngin
    spec:
      containers:
      - image: localhost:5000/database-enterprise:12.2.0.1
        name: ora12-container
        env:
        - name: DB_SID
          value: ORCLCDB
        - name: DB_PDB
          value: ORCLPDB
        - name: ORACLE_PWD
          value: dada
        - name: ORACLE_CHARACTERSET
          value: AL32UTF8
        ports:
        - containerPort: 1521
          name: ora12-lsnr
        - containerPort: 5500
          name: ora12-em
        volumeMounts:
        - name: ora12-persistent-storage
          mountPath: /ORCL
      volumes:
      - name: ora12-persistent-storage
        persistentVolumeClaim:
          claimName: ora12-pvc

EOF

kubectl apply -f ora12-svc.yaml
````
#Cleanup
````bash
kubectl delete svc ora12-svc
kubectl delete deploy ora12-deploy
kubectl delete pvc ora12-pvc
kubectl delete pv ora12-pv
````