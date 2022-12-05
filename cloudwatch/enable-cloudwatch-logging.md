# Cloudwatch logging of an EKS cluster

## enable e.g. via yaml config file

```bash
eksctl utils update-cluster-logging --config-file eks-course.yaml --approve
```

## disable via plain commandline call

```bash
eksctl utils update-cluster-logging --name=EKS-course-cluster --disable-types all
```
