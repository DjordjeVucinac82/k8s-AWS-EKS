## Prerequisites

Create role for eks wih eksClusterPolicy \
User with admin priv

### Install aws cli, aws --version

### install eksctl, eksctl version

### install kubectl, kubectl version --short --client

TODO
<https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/>

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
<https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer>

# Kubernetes Dashboard

## Metric Server
<https://github.com/kubernetes-sigs/metrics-server> \
<https://artifacthub.io/packages/helm/metrics-server/metrics-server>

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml` \
`serviceaccount/metrics-server created` \
kubectl get deployment metrics-server -n kube-system

## Dashboard

<https://github.com/kubernetes/dashboard/releases> \
<https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md> \
<https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/> \
<https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html> \
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

## Enable EFS

Go to AWS Console -> EFS service, and make new efs, name aws-efs, all AZ's \
Take FilesystemId and DNS Name from EFS, go into provisioner! \
*specify the security group of your EC2-worker-nodes, to be applied to EFS as well*
install the package *amazon-efs-utils* on all worker nodes \
ssh -i k8s-eks.pem ec2-user@18.197.34.156 "sudo yum install -y amazon-efs-utils"

### create namespaces

`kubectl create namespace eks-wp-efs`

### Creatte Storage

https://www.linkedin.com/pulse/deploy-blog-site-aws-eks-using-efs-provisioner-shubham-rasal/ \

`kubectl apply -f wordpress-efs/create-efs-provisioner.yaml --namespace=eks-wp-efs` \
`kubectl apply -f wordpress-efs/create-rbac.yaml --namespace=eks-wp-efs` \
`kubectl apply -f wordpress-efs/create-storage.yaml --namespace=eks-wp-efs` \
Missing DNS_NAME var in efs-provisioner!!! https://github.com/kubernetes-retired/external-storage/blob/master/aws/efs/deploy/deployment.yaml \
ssh into ec2 with efs and execute `sudo mount | grep amazonaws` \
`sudo ls -lha /var/lib/kubelet/pods/630e3489-ea9d-4b66-adcb-6163e4e2d4c9/volumes/kubernetes.io~nfs/pv-volume`

### Deploy MySQL
Create mysql password MYSQL_ROOT_PASSWORD \
`kubectl create secret generic mysql-pass --from-literal=password=eks-mysql-Pa$$word34 --namespace=eks-wp-efs` \
`kubectl get secrets --namespace=eks-wp-efs`
`kubectl apply -f wordpress-efs/deploy-mysql.yaml --namespace=eks-wp-efs`
`kubectl describe pod wordpress-mysql-6bf49d7b9-dzt2v --namespace=eks-wp-efs`


### Deploy Wordpress
https://codex.wordpress.org/WordPress_Versions \
https://v1-22.docs.kubernetes.io/docs/concepts/services-networking/service/#loadbalancer \
version 6.1.1 latest on date 06.12.2022 \
Deploy wordpress app \
https://hub.docker.com/_/wordpress \
`kubectl apply -f wordpress-efs/deploy-wordpress.yaml --namespace=eks-wp-efs` \
`kubectl describe service wordpress --namespace=eks-wp-efs | grep Ingress` \
`kubectl exec wordpress-67ddf79c57-9cmrb mount --namespace=eks-wp-efs` \
output: fs-0f2d67d51c904ca13.efs.eu-central-1.amazonaws.com:/efs-wordpress-pvc-cc2cbf03-3df6-44b4-a4fe-829e679b761c on /var/www/html type nfs4
`kubectl describe service wordpress --namespace=eks-wp-efs | grep Ingress`

!!!TODO!!!
Install Prometheus and Grafana Dashboard manifest, 
Install via helm charts!!!
