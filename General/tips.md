````bash
# Alias
alias k='kubectl'

# Get short names for resources
#   pod        : po
#   namespaces : ns
#   deployments: deploy
#   configmaps : cm
#   services   : svc
k api-resources

# Fast Delete objects
kubectl delete pod nginx --grace-period=0 --force


````