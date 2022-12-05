# Providing RBAC to IAM users

## add a cluster admin

Steps:

1. create IAM user in AWS console (_k8s-cluster-admin_)
2. create access key for this user and store it locally
3. add user to configmap aws-auth
4. add user+accesskey to aws credentials file in dedicated section (profile)

Execution:

fetch current configmap before adding our user mapping

```bash
kubectl -n kube-system get cm
kubectl -n kube-system get configmap aws-auth -o yaml > aws-auth-configmap.yaml

```

#### 44 edit the yaml file and add a "mapUsers" section

```bash
  mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-cluster-admin
      username: k8s-cluster-admin
      groups:
        - system:masters
Save and quit        
kubectl apply -f aws-auth-configmap.yaml -n kube-system  
kubectl -n kube-system describe cm aws-auth      
```

add user to ~/.aws/credentials by creating a new section

```bash
[clusteradmin]
aws_access_key_id=.....
aws_secret_access_key=.....
region=us-east-1
output=json
```

check which user is currently active

```bash
aws sts get-caller-identity
export AWS_PROFILE="clusteradmin"
aws sts get-caller-identity
```

## add read-only user for particular namespace

```bash
kubectl create namespace production
```

create IAM user and grab access-key  user name: prod-viewer, programatic, .scv

create role

```bash
# Make new file role.yml and paste below code
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: production
  name: prod-viewer-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]  # can be further limited, e.g. ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch"] 
```

create rolebinding

```bash
# Make file rolebinding.yml and paste code beloww -> check apiVersion
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: prod-viewer-binding
  namespace: production
subjects:
- kind: User
  name: prod-viewer
  apiGroup: ""
roleRef:
  kind: Role
  name: prod-viewer-role
  apiGroup: ""
```

create role and binding

```bash
kubectl apply -f role.yaml rolebinding.yaml
```

add user to aws-auth configmap

Go check 44 up in file, need to add mapUsers (prod-viewer) to config map

```bash
  mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/prod-viewer
      username: prod-viewer
      groups:
        - prod-viewer-role        
```

kubecl apply -f aws-auth-configmap.yaml
add user to ~/.aws/credentials file

```bash
[clusteradmin]
aws_access_key_id=
aws_secret_access_key=
region=us-east-1
output=json
[productionviewer]
aws_access_key_id=
aws_secret_access_key=
region=us-east-1
output=json
```

set this user as the active one

```bash
aws sts get-caller-identity
export AWS_PROFILE="productionviewer"
aws sts get-caller-identity
```
