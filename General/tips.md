# Enable autocompletion
````bash
source <(kubectl completion bash)
````

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

alias kgetpo='kubectl get po -o=wide'
# Fast Delete objects kubectl delete pod nginx --grace-period=0 --force
alias kfdelpo='function _kfdelpo(){ kubectl delete po $1 --grace-period=0 --force; };_kfdelpo'
# Set namespace from $1
alias ksetns='function _ksetns() { kubectl config set-context --current --namespace=$1; }; _ksetns'
# Get namespace
alias kgetns='kubectl config view --minify | grep namespace:'
# Run pod : pod name $1 image $2
alias krun='function _krun() { kubectl run $1 --image="$2"; }; _krun'
# Run pod no restart: pod name $1 image $2
alias krun-nr='function _krun_nr() { kubectl run $1 --image="$2" --restart="Never; }; _krun_nr'

````

##Setting the namespace preference
kubectl config set-context --current --namespace=ckad
OR
kubectl config set-context $(kubectl config current-context) --namespace=labels

# Validate it
kubectl config view --minify | grep namespace:

alias kfdelpo='function _kfdelpo(){ kubectl delete po $1 --grace-period=0 --force; };_kfdelpo'