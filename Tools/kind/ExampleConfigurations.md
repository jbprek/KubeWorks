# Example configurations

## 1. WSL2 CKAD single node 5 ports [See Ref doc](https://kind.sigs.k8s.io/docs/user/using-wsl2/)

````yaml
# kind-config.yaml
cat EOF << > 
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
````


````bash
# Create Cluster
kind create cluster --config=kind-config.yaml 
# delete Cluster
kind delete clusters kind
````


# Kind 5 Nodeports, 5 Volumes, Ingress
````bash
cat << EOF > kind-config-55.yaml 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
# 5 Port for Node mapping  
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
    - hostPath: /home/john/my-kind/vol5
      containerPath: /vol5
EOF
    
kind create cluster --config  kind-config-ingress.yaml 
# From https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s

````
# Kind 5 Nodeports, 5 Volumes, Ingress
````bash
cat << EOF > kind-config-ingress.yaml 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
# 2 Ports for Ingress 
    - containerPort: 80
      hostPort: 10080
      protocol: TCP  
    - containerPort: 443
      hostPort: 10443
      protocol: TCP  
# 5 Port for Node mapping  
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
    - hostPath: /home/john/my-kind/vol5
      containerPath: /vol5
EOF
    
kind create cluster --config  kind-config-ingress.yaml 
# From https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s

````

# Kind Local Registy, 5 Nodeports, 5 Volumes
````sh
#!/bin/sh
set -o errexit

for i in {1..5}
do 
  mkdir -p /home/john/my-kind/vol$i
done  
  

# create registry container unless it already exists
reg_name='kind-registry'
reg_port='5000'
running="$(docker inspect -f '{{.State.Running}}' "${reg_name}" 2>/dev/null || true)"
if [ "${running}" != 'true' ]; then
  docker run \
    -d --restart=always -p "127.0.0.1:${reg_port}:5000" --name "${reg_name}" \
    registry:2
fi

# create a cluster with the local registry enabled in containerd
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:${reg_port}"]
    endpoint = ["http://${reg_name}:${reg_port}"]
nodes:
- role: control-plane
  extraPortMappings:
    # Five ports for NodePorts
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
EOF

# connect the registry to the cluster network
# (the network may already be connected)
docker network connect "kind" "${reg_name}" || true

# Document the local registry
# https://github.com/kubernetes/enhancements/tree/master/keps/sig-cluster-lifecycle/generic/1755-communicating-a-local-registry
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: local-registry-hosting
  namespace: kube-public
data:
  localRegistryHosting.v1: |
    host: "localhost:${reg_port}"
    help: "https://kind.sigs.k8s.io/docs/user/local-registry/"
EOF
    
````