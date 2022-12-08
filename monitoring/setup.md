# Installing Prometheus & Grafana

## Prometheus
Since eksctl out of the box adds storage class "gp2" we will use this for Prometheus data storage.  
In real world scenarios you may use IO optmized storage, like e.g. _io1_ , for better performance.

```bash
kubectl create namespace prometheus
helm install prometheus stable/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

check:  
```kubectl get svc -n prometheus```
...and grab the port from the output !

set port forwarding to access Prometheus UI:

```bash
kubectl port-forward svc/prometheus-server 8888:80
```

access Prometheus UI from your *local* browser:
* http://localhost:8888

## Grafana

### installation

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
```kubectl get svc - grafana grafana```

### import dashboard(s)

* goto https://grafana.com/grafana/dashboards and select the *ID* of the dashboard you want to import  
* in Grafana click on *+* , click *Import* and put the *ID* from the previous step into the text field