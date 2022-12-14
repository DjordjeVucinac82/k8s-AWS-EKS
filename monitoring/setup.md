# Installing Prometheus & Grafana

2 in 1 https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack
## Prometheus
Since eksctl out of the box adds storage class "gp2" we will use this for Prometheus data storage.  
In real world scenarios you may use IO optmized storage, like e.g. _io1_ , for better performance.

#### OLD/stable
```bash
kubectl create namespace prometheus
helm install prometheus stable/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

#### NEW/prometheus-comunity
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace prometheus
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

https://stackoverflow.com/questions/72126048/circleci-message-error-exec-plugin-invalid-apiversion-client-authentication \
I'll install prometheus-community version! \
Update aws-cli version and then update kubeconfig \
`aws eks update-kubeconfig --name EKS-cluster-basic --region eu-central-1` \
`export POD_NAME=$(kubectl get pods --namespace prometheus -l "app=prometheus-pushgateway,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")` \
`kubectl --namespace prometheus port-forward $POD_NAME 9090` \
`localhost:9090` \
Go to graph and put kube_node_info ->execute

OLD!
check:  
```kubectl get svc -n prometheus```
...and grab the port from the output !
set port forwarding to access Prometheus UI:
```bash
kubectl port-forward svc/prometheus-server 9090:80
```
access Prometheus UI from your *local* browser:
* http://localhost:9090

## Grafana

### installation

#### NEW /grafana
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
kubectl create namespace grafana
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword='GrafanaAdm!n' \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer
```
#### OLD /stable
```bash
kubectl create namespace grafana
helm install grafana stable/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword='GrafanaAdm!n' \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer
```

get admin password for logging in

```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

open Grafana

grab the ELB URL from the Grafana Loadbalancer, and open it in the browser
```kubectl get svc -n grafana grafana```

### import dashboard(s)

In grafana dashboard go to Configuration -> Data sources and check for Prometheus instance

* go to https://grafana.com/grafana/dashboards , search for 'Cluster Monitoring for Kubernetes' (ID: 10000) nad kube-state-metrics-v2 (13332) , and select the *ID* of the dashboard you want to import  
* in Grafana click on *+*  Create Dashboards, click *Import* and put the *ID* from the previous step into the text field. Befor importing, check: Unique Identifier (UID)=JABGX_-mz and check Prometheus under prometheus -> Import !!!




