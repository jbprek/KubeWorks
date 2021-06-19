# Example configurations

## 1. WSL2 CKAD single node 5 ports [See Ref doc](https://kind.sigs.k8s.io/docs/user/using-wsl2/)

````yaml
# ckad-config.yaml
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

Run command

````
kind create cluster --config=ckad-config.yaml --name ckad
````