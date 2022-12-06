## Prerequisites

Create role for eks wih eksClusterPolicy \
User with admin priv

### Install aws cli, aws --version

### install eksctl, eksctl version

### install kubectl, kubectl version --short --client

TODO
https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/ 
### Create eks cluster 

`eksctl create cluster -f cluster/eks-cluster.yml` \
`kubectl get nodes` \
`eksctl get nodegroup --cluster EKS-cluster-basic` \
`eksctl scale nodegroup --cluster=EKS-cluster-basic --node=5 --name=ng1`

### Update nodegroup 

`eksctl create nodegroup --config-file=cluster/eks-cluster-spot-mix.yml --include='ng-mixed'` \
`eksctl delete nodegroup --config-file=cluster/eks-cluster-spot-mix.yml --include='ng-mixed' --approve`

### Scaler 

`eksctl create nodegroup --config-file=cluster/eks-cluster-scaler.yml` \
`eksctl delete nodegroup -f cluster/eks-cluster-scaler.yml --include='ng-1' --approve`

## Apply autoscaler

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml` \
serviceaccount/cluster-autoscaler created \
clusterrole.rbac.authorization.k8s.io/cluster-autoscaler created \
role.rbac.authorization.k8s.io/cluster-autoscaler created \
clusterrolebinding.rbac.authorization.k8s.io/cluster-autoscaler created \
rolebinding.rbac.authorization.k8s.io/cluster-autoscaler created \
deployment.apps/cluster-autoscaler created

Put required annotation to the deployment: \
`kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/ \ safe-to-evict="false" --overwrite` \

edit deployment and set your EKS cluster name \
`kubectl -n kube-system edit deployment.apps/cluster-autoscaler` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler` \
`kubectl -n kube-system describe deployment cluster-autoscaler`

### test the autoscaler

create a deployment of nginx \
`kubectl apply -f cluster/nginx-deployment.yml` \
`kubectl get pod --all-namespaces -o wide` \
`kubectl get nodes -l instance-type=spot` \
`kubectl scale --replicas=3 deployment/test-autoscaler` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"` \
`kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"`

### Cloudwatch is in folder
path: cloudwatch/container-insights-logging.md

### Helm

`helm repo add stable https://charts.helm.sh/stable --force-update` \
`helm version` , <https://artifacthub.io/> \
`helm repo list` \
`helm repo update` \
`helm install redis-test stable/redis`

### IAM users

path: cloudwatch/iam-user.md 

#### Network security Calico - optional
## Loadbalancer
https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer

# Kubernetes Dashboard

## Metric Server
https://github.com/kubernetes-sigs/metrics-server \
https://artifacthub.io/packages/helm/metrics-server/metrics-server 

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml` \
`serviceaccount/metrics-server created` \
kubectl get deployment metrics-server -n kube-system

## Dashboard

https://github.com/kubernetes/dashboard/releases \
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md \
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ \
https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html \
Change DASHBOARD_RELEASE or export, export DASHBOARD_RELEASE=v2.4.0 \
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml` \
namespace/kubernetes-dashboard created \

`kubectl apply -f dashboard-k8s/admin-service-account.yaml` \

`kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')` \
copy token! \
`kubectl -n kubernetes-dashboard create token admin-user` neradi \

`kubectl proxy` \

open browser at `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login` \
enter token!

# Stateless PHP-Redis

### Redis

Deployment redis master and service \
`kubectl apply -f stateless-php/redis-master.yaml` \
`kubectl get pods` \
`kubectl get services` \
Deployment redis slaves and service \
`kubectl apply -f stateless-php/redis-slaves.yaml` \
`kubectl delete -f stateless-php/redis-master.yaml` \
`k`ubectl delete -f stateless-php/redis-slaves.yaml`

### PHP

Deployment frontend-php app and service \
`kubectl apply -f stateless-php/frontend.yaml` \
`kubectl get pods -l app=guestbook -l tier=frontend` \
grab the public DNS (EXTERNAL-IP) of the frontend service LoadBalancer (ELB): \
`kubectl ge service frontend` \
`kubectl describe service frontend` LoadBalancer Ingress:___ \
`kubectl delete -f stateless-php/frontend.yaml` 

#### Scaling
`kubectl scale --replicas 5 deployment frontend` \
`kubectl get rs` \


# Wordpress-EFS











