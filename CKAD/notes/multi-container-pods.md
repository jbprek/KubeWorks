# Multi-Container Pods

- Two use cases
1. Init containers
2. Helper containers (sidecar, adapter, ambassador patterns)

## Init containers
Init containers are not meant to keep running over a longer period of time.
### Example  A Pod defining an init container

1. define descriptor
````yaml
# Fixed ..initContainers.args
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  initContainers:
    - name: configurer
      image: busybox:1.32.0
      args: ['/bin/sh', '-c', 'echo Configuring application... &&
              mkdir -p /usr/shared/app && echo -e "{\"dbConfig\": {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" > /usr/shared/app/config.json']
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

3. Inspect logs of (init) containers --container (-c) arg
````
kubectl logs business-app -c configurer
kubectl logs business-app --container configurer
# Also works for the web app
kubectl logs business-app --container web
# In our case the following is enough
kubectl logs business-app

````

## Sidecar Pattern
Typically, there are two different categories of containers: the container that runs the application and another container that provides helper functionality to the primary application. In the Kubernetes space, the container providing helper functionality is called a sidecar. Among the most commonly used capabilities of a sidecar container are file synchronization, logging, and watcher capabilities. The sidecars are not part of the main traffic or API of the primary application. They usually operate asynchronously and are not involved in the public API.

````yaml
#sidecar.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  - name: sidecar
    image: busybox
    command: ["sh","-c","while true; do if [ \"$(cat /var/log/nginx/error.log \
              | grep 'error')\" != \"\" ]; then echo 'Error discovered!'; fi; \
              sleep 10; done"]
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  volumes:
  - name: logs-vol
    emptyDir: {}
````

2. Create Pod
````
kubectl apply  -f sidecar.yaml
````

3. Ops 
- Inspect  logs
````
# Sidecar
kubectl logs  webserver -c sidecar
# nginx
kubectl logs  webserver -c nginx
````

- Check IP address 
````
$ kubectl get po -o wide
NAME        READY   STATUS    RESTARTS   AGE    IP            NODE                 NOMINATED NODE   READINESS GATES
webserver   2/2     Running   0          4m4s   10.244.0.23   ckad-control-plane   <none>           <none>
````
- Make an HTTP request
````
kubectl run  bs --image busybox --restart=Never -it --rm --command -- wget -O- http://10.244.0.23
````
- Create shell in sidecar to issue http requests
````
kubectl exec webserver -it -c sidecar -- /bin/sh
# Home Page
# wget -O- localhost
# Http 404 W will cause error to be logged from sidecar
# wget -O- localhost/does-not-exist 
````

- Wrong url will cause error to be logged from sidecar