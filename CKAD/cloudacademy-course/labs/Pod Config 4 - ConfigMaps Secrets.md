Introduction
ConfigMaps are a type of Kubernetes Resource that is used to decouple configuration artifacts from container image content to keep containerized applications portable. The configuration data is stored as key-value pairs. One ConfigMap may contain one or more key-value pairs. With kubectl, ConfigMaps can be created from:

- Environment variable files consisting of key-value pairs separated by equal signs, e.g. key=value. The file should have one key-value pair per line.
- Regular files or directories of files result in keys that are the names of the files and values that are the contents of the files.
- Literals consisting of individual key-value pairs that you specify on the command line.
- Writing a YAML manifest file of kind: ConfigMap.

ConfigMaps can be mounted into containers as volumes or as environment variables. Like most Kubernetes Resources, ConfigMaps are namespaced so only Pods in the same Namespace as a ConfigMap can use the ConfigMap.

In this Lab Step, you will see an example of how to create a ConfigMap from a literal key-value pair and mount the configuration data into a Pod using a volume.

````bash
# 0. Load kubectl shell completions for your current shell session:
source <(kubectl completion bash)

#1. 1. Create a Namespace for the resources you'll create in this Lab Step and change your default kubectl context to use the Namespace:
# Create namespace
kubectl create namespace configmaps
# Set namespace as the default for the current context
kubectl config set-context --current --namespace=configmaps
#Verify
kubectl config view | grep namespace:

#2. Create a ConfigMap from two literal key-value pairs:
kubectl create configmap app-config --from-literal=DB_NAME=testdb \
  --from-literal=COLLECTION_NAME=messages
  
#3. Display the ConfigMap:
kubectl get configmaps app-config -o yaml  
OR
kubectl describe cm app-config

#. Create a Pod that mounts the ConfigMap using a volume:
cat << 'EOF' > pod-configmap.yaml
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
    - name: config
      mountPath: /config
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: config
    # Declare the configMap to use for the volume
    configMap:
      name: app-config
EOF
kubectl create -f pod-configmap.yaml
#5. List the /config directory, where the ConfigMap volume is mounted, in the container:
kubectl exec db -it -- ls /config
#6. Get the contents of the DB_NAME file
kubectl exec db -it -- cat /config/DB_NAME && echo

````

Introduction
ConfigMaps are a type of Kubernetes Resource that is used to decouple configuration artifacts from container image content to keep containerized applications portable. The configuration data is stored as key-value pairs. One ConfigMap may contain one or more key-value pairs. With kubectl, ConfigMaps can be created from:

- Environment variable files consisting of key-value pairs separated by equal signs, e.g. key=value. The file should have one key-value pair per line.
- Regular files or directories of files result in keys that are the names of the files and values that are the contents of the files.
- Literals consisting of individual key-value pairs that you specify on the command line.
- Writing a YAML manifest file of kind: ConfigMap.

ConfigMaps can be mounted into containers as volumes or as environment variables. Like most Kubernetes Resources, ConfigMaps are namespaced so only Pods in the same Namespace as a ConfigMap can use the ConfigMap.

In this Lab Step, you will see an example of how to create a ConfigMap from a literal key-value pair and mount the configuration data into a Pod using a volume.

````bash


#1. 1. Create a Namespace for the resources you'll create in this Lab Step and change your default kubectl context to use the Namespace:
# Create namespace
kubectl create namespace secrets
# Set namespace as the default for the current context
kubectl config set-context --current --namespace=secrets
#Verify
kubectl config view | grep namespace:

#2. Use kubectl to create a Secret named app-secret:
kubectl create secret generic app-secret --from-literal=password=123457
#3. Get the YAML output for the Secret you created:
kubectl get secret app-secret -o yaml
#4. Confirm the secret value is base-64 encoded by decoding it:
#The base64 command can encode/decode strings. 
# The --decode option must be specified to decode while the behavior with no options is to encode. 
# The final echo is used to add a new line 
kubectl get secret app-secret -o jsonpath="{.data.password}" \
  | base64 --decode \
  && echo
  
#5. Create a Pod that uses the Secret through an environment variable:
#When using a secret through an environment variable, you must include valueFrom.secretKeyRef to specify the source of the environment variable.
cat << EOF > pod-secret.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
spec:
  containers:
  - image: busybox:1.30.1
    name: busybox
    args:
    - sleep
    - "3600"
    env:
    - name: PASSWORD      # Name of environment variable
      valueFrom:
        secretKeyRef:
          name: app-secret  # Name of secret
          key: password     # Name of secret key
EOF

kubectl create -f pod-secret.yaml
  
````