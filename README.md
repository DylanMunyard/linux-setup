# linux-setup

## AWS MFA
`git clone https://github.com/asagage/aws-mfa-script.git`

Add to `~/.bashrc` to create an alias:

`source /home/dylan/Documents/aws-mfa-script/alias.sh`

Usage: `mfa <token>`

## kube / k3s
kubectl

`snap install kubectl --classic`

k3s

`curl -sfL https://get.k3s.io | sh -`

By default requires sudo, to fix add `K3S_KUBECONFIG_MODE="644"` to 
`/etc/systemd/system/k3s.service.env`.

Add aliases:
```
#kubectl autocomplete
source <(kubectl completion bash)

#Exec into a C2 pod
c2exec() {
	namespace=${2:-saas-dev}
	kubectl exec -it $(kubectl get pod --selector=app=$1 -n $namespace -o jsonpath='{.items[*].metadata.name}') -n $namespace -- powershell
}
alias c2exec=setToken

#k3s
alias k3ktl='k3s kubectl'
alias k3proxy='sudo k3s kubectl proxy'
complete -F __start_kubectl k3ktl
```

Run `k3proxy` then accesses services `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`

Start k3s `systemctl start k3s` 

Stop k3s `systemctl stop k3s`
