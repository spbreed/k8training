# Shell
### Delete:
- Shift -> D delete line
- D -> g delete after cursor
- :g/^$/d -> empty lines

- grep -A 5

### Autocomplete:
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

## Shorthand

alias kk='kubectl'
alias kkgd='kubectl get deployments'
alias kkgp='kubectl get pods'
alias kkgj='kubectl get jobs'
alias kkgs='kubectl get service'
alias kkds='kubectl delete service'
alias kkgl='kubectl logs'
alias kkglf='kubectl logs -f'
alias kkgpvc='kubectl get pvc'
alias kkgpv='kubectl get pv'
alias kkdd='kubectl delete deployments'
alias kkdp='kubectl delete pods'
alias kkdj='kubectl delete jobs'
alias kkdpv='kubectl delete pv'
alias kkdpvc='kubectl delete pvc'
alias kkgpn='kubectl get pods -o wide --sort-by="{.spec.nodeName}"'
alias kkdcp='kubectl describe pod'
alias kkdcs='kubectl describe service'
alias kkdcpvc='kubectl describe pvc'
alias kkdcpv='kubectl describe pv'
alias kkgn='kubectl get nodes'
alias kka='kubectl apply -f'
alias kkdf='kubectl delete -f'
alias kkcc='kubectl config current-context'
alias kkx='kubectl exec -it'
alias kkr='kubectl run -it --rm --restart=Never'
alias kkuc='kubectl config use-context'
alias kkcd='kubectl create deployment'
alias kkcf='kubectl create -f'
alias kktn='kubectl taint nodes --all'
alias kkdn=' kubectl describe node'
alias kkcn='kubectl create namespace'
alias kkrh='kubectl rollout history'
alias kkcurl='kubectl run -it --rm --restart=Never --image=radial/busyboxplus -- sh'


## paste:
:set paste

### tmux
- ctl + b -> " horizonal
- ctl + b -> % vertical
- ctl + b -> up 

### Autocomplete:
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

### troubleshoot services
- Start service:
- systemctl | grep kub
- dpkg -l | grep docker