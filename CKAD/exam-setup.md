vi .vimrc

set ts=2
set sw=2
set expandtab
set paste

source <(kubectl completion bash)
alias kn='kubectl config set-context --current --namespace '
export do="--dry-run=client -o yaml"
export rm="--restart=Never --rm -i"
export kp="--force --grace-period=0"