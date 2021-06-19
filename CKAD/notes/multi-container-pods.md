# Multi-Container Pods

- Two use cases
1. Init containers
2. Helper containers (sidecar, adapter, ambassador patterns)

## Init containers
### Example  A Pod defining an init container

1. define descriptor
````yaml
# init.yaml
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  initContainers:
  - name: configurer
    image: busybox:latest
    command: ['sh', '-c', 'echo Configuring application... && \
              mkdir -p /usr/shared/app && echo -e "{\"dbConfig\": \
              {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" \
              > /usr/shared/app/config.json']
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  volumes:
  - name: configdir
    emptyDir: {}
````
2. Create Pod
````
kubectl create -f init.yaml
````

3. Inspect logs of individual containers --container arg
````
kubectl logs business-app -c configurer
````