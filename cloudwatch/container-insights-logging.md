# Container insights with Cloudwatch Metrics

## add policy to your nodegroup(s)

add policy *CloudWatchAgentServerPolicy* to nodegroup(s) role

## deploy the cloudwatch agent

runs as daemonset, means one per node

```bash
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/EKS-course-cluster/;s/{{region_name}}/us-east-1/" | kubectl apply -f -

```

## load generation

```bash
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --limits=cpu=500m --expose --port=80
```

```bash
kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh

Hit enter for command prompt

while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```

# Cloudwatch logging of an EKS cluster

## enable e.g. via yaml config file

```bash
eksctl utils update-cluster-logging --config-file eks-course.yaml --approve
```

## disable via plain commandline call

```bash
eksctl utils update-cluster-logging --name=EKS-course-cluster --disable-types all
```
