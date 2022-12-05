Create role for eks wih eksClusterPolicy
User with admin priv
## Prerequisites

### Install aws cli, aws --version
### install eksctl, eksctl version
### install kubectl, kubectl version --short --client
TODO
### Create eks cluster \
`eksctl create cluster -f cluster/eks-cluster.yml` \
`kubectl get nodes` \ 
`eksctl get nodegroup --cluster EKS-cluster-basic` \
`eksctl scale nodegroup --cluster=EKS-cluster-basic --node=5 --name=ng1` \

### Update nodegroup \
`eksctl create nodegroup --config-file=cluster/eks-cluster-spot-mix.yml --include='ng-mixed'` \
`eksctl delete nodegroup --config-file=cluster/eks-cluster-spot-mix.yml --include='ng-mixed' --approve` \

### Scaler \
`eksctl create nodegroup --config-file=cluster/eks-cluster-scaler.yml` \
`eksctl delete nodegroup -f cluster/eks-cluster-scaler.yml --include='ng-1' --approve` \

## Apply autoscaler
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml` \
serviceaccount/cluster-autoscaler created \
clusterrole.rbac.authorization.k8s.io/cluster-autoscaler created \
role.rbac.authorization.k8s.io/cluster-autoscaler created \
clusterrolebinding.rbac.authorization.k8s.io/cluster-autoscaler created \
rolebinding.rbac.authorization.k8s.io/cluster-autoscaler created \
deployment.apps/cluster-autoscaler created \

Put required annotation to the deployment: \
`kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/ \ safe-to-evict="false" --overwrite` \

edit deployment and set your EKS cluster name \
`kubectl -n kube-system edit deployment.apps/cluster-autoscaler` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler` \
`kubectl -n kube-system describe deployment cluster-autoscaler` \

### test the autoscaler
create a deployment of nginx \
`kubectl apply -f cluster/nginx-deployment.yml` \
`kubectl get pod --all-namespaces -o wide` \
`kubectl get nodes -l instance-type=spot` \
`kubectl scale --replicas=3 deployment/test-autoscaler` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"` \

### Cloudwatch is in folder

### Helm
`helm repo add stable https://charts.helm.sh/stable` \
`helm version` \
`helm repo list` \
`helm repo update` \
`helm install redis-test stable/redis` \