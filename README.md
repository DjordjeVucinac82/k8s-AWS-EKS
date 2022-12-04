# k8s-AWS-EKS
Kubernetes running Wordpress and suff:)


apiVersion: extensions/v1beta1 is depricated Deployment, use apps/v1 and add selector:
```bash
spec:
  selector:
    matchLabels:
      app: nginx  
```      