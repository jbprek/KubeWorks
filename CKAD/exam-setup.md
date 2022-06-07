```bash
vi .vimrc

set ts=2
set sw=2
set expandtab

source <(kubectl completion bash)
source <(helm completion bash)
alias ksn='kubectl config set-context --current --namespace '
alias kgn='kubectl config view | grep namespace'
alias kgp='kubectl get po -o wide --show-labels'
alias kga='kubectl get all'
alias kgaw='kubectl get all -o wide --show-labels'
export do='--dry-run=client -o yaml'
export rm='--restart=Never --rm'

```