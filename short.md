## Minimum instalation Dashboard

#### Create eks cluster with 2 node group t2.small, eu-central-1
`eksctl create cluster -f cluster/eks-cluster.yml` \
Insall auto-scaler \
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml` \
Put required annotation to the deployment: \
`kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false" --overwrite` \
Edit deployment and set your EKS cluster name \
`kubectl -n kube-system edit deployment.apps/cluster-autoscaler` \
https://github.com/kubernetes/autoscaler/releases \
Change under spec ->containers ->command -> --node-group-auto-discovery= put ur cluster name in the end of line EKS-cluster-basic \
Change image option, put autoscaler version {k8s.grc.io/autoscaling/cluster-autoscaler:v1.22.2}

https://github.com/kubernetes-sigs/metrics-server \
https://artifacthub.io/packages/helm/metrics-server/metrics-server \
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

#### Install kubernetes dashboard 
`kubectl create namespace kubernetes-dashboard` \
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml --namespace=kubernetes-dashboard` \
Creae admin user in dashboard \
`kubectl apply -f dashboard-k8s/admin-service-account.yaml --namespace=kubernetes-dashboard` \
Copy dashboard token \
`kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')`
Start kubernetes dashboard \
`kubectl proxy` \
Open browser at `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login`
enter token!
