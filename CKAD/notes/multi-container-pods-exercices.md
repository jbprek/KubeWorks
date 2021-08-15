# Multi-Container Pods Exercises
##dgkanatsios
1. Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'
````
kubectl run busy1 --image=busybox --restart=Never --dry-run=client -o yaml --command --  /bin/sh -c 'mkdir -p /usr/shared/app ; echo -e "{\"dbConfig\": {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" > /usr/shared/app/config.json; sleep 60'
kubectl run busy1 --image=busybox --restart=Never  --command --  /bin/sh -c 'mkdir -p /usr/shared/app ; echo -e "{\"dbConfig\": {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" > /usr/shared/app/config.json; sleep 60'
 ````
- Busybox running command 
2. Create pod with nginx container exposed at port 80. 
   Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". 
   Make a volume of type emptyDir and mount it in both containers. 
   For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". 
   When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"


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
    command: ['/bin/sh', '-c', 'echo Configuring application... && \
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