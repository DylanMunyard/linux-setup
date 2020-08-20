# linux-setup

## AWS MFA
`git clone https://github.com/asagage/aws-mfa-script.git`

Add to `~/.bashrc` to create an alias:

`source /home/dylan/Documents/aws-mfa-script/alias.sh`

`aws configure` and set up the access key from EC2 console. 
Then copy ~/.aws/config and add
```
[profile reset]
aws_access_key_id=access_key_id
aws_secret_access_key=secret_access_key
region = ap-southeast-2
output = json
```

Modify ~/Documents/mfa.sh:
```
#!/bin/bash
#
# Sample for getting temp session token from AWS STS
#
# aws --profile youriamuser sts get-session-token --duration 3600 \
# --serial-number arn:aws:iam::012345678901:mfa/user --token-code 012345
#
# Once the temp token is obtained, you'll need to feed the following environment
# variables to the aws-cli:
#
# export AWS_ACCESS_KEY_ID='KEY'
# export AWS_SECRET_ACCESS_KEY='SECRET'
# export AWS_SESSION_TOKEN='TOKEN'

AWS_CLI=`which aws`

if [ $? -ne 0 ]; then
  echo "AWS CLI is not installed; exiting"
  exit 1
else
  echo "Using AWS CLI found at $AWS_CLI"
fi

# 1 or 2 args ok
if [[ $# -ne 1 && $# -ne 2 ]]; then
  echo "Usage: $0 <MFA_TOKEN_CODE> <AWS_CLI_PROFILE>"
  echo "Where:"
  echo "   <MFA_TOKEN_CODE> = Code from virtual MFA device"
  echo "   <AWS_CLI_PROFILE> = aws-cli profile usually in $HOME/.aws/config"
  exit 2
fi

echo "Reading config..."
if [ ! -r ~/Documents/aws-mfa-script/mfa.cfg ]; then
  echo "No config found.  Please create your mfa.cfg.  See README.txt for more info."
  exit 2
fi

echo "Removing credentials"
rm -f ~/.aws/credentials

AWS_CLI_PROFILE=${2:-reset}
MFA_TOKEN_CODE=$1
ARN_OF_MFA=$(grep "^$AWS_CLI_PROFILE" ~/Documents/aws-mfa-script/mfa.cfg | cut -d '=' -f2- | tr -d '"')

echo "AWS-CLI Profile: $AWS_CLI_PROFILE"
echo "MFA ARN: $ARN_OF_MFA"
echo "MFA Token Code: $MFA_TOKEN_CODE"

echo "Your Temporary Creds:"
aws --profile $AWS_CLI_PROFILE sts get-session-token --duration 129600 \
  --serial-number $ARN_OF_MFA --token-code $MFA_TOKEN_CODE --output text \
  | awk '{printf("export AWS_ACCESS_KEY_ID=\"%s\"\nexport AWS_SECRET_ACCESS_KEY=\"%s\"\nexport AWS_SESSION_TOKEN=\"%s\"\nexport AWS_SECURITY_TOKEN=\"%s\"\n",$2,$4,$5,$5)}' | tee ~/Documents/aws-mfa-script/.token_file

aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID 
aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
aws configure set aws_session_token $AWS_SESSION_TOKEN
aws configure set aws_security_token $AWS_SECURITY_TOKEN
```

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
