# A Practical Guide to EKS

In this repository you will find all the assets required for the course `A Practical Guide to Amazon EKS`, by A Cloud Guru.


## Bookstore application

This solution has been built for for explaining all the concepts in this course. It is complete enough for covering a real case of microservices running on EKS and integrating with other AWS Services.

> You can find in [here](_docs/api.md) the documentation of the APIs.


## Running on Local

To Run application on local with pointing to DynamoDB in your AWS account to store data, run following. 
```
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>

# Create DynamoDB tables
aws cloudformation create-stack --stack-name dev-eks-bookstore-client-api-table --template-body file://$PWD/clients-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-eks-bookstore-inventory-api-table --template-body file://$PWD/inventory-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-eks-bookstore-renting-api-table --template-body file://$PWD/renting-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-eks-bookstore--resource-api-table --template-body file://$PWD/resource-api/infra/cloudformation/dynamodb-table.json

# Start Docker Containers
docker-compose up -d
```

> Open Browser and hit url http://localhost

### Cleanig Resources
```
docker-compose down

aws cloudformation list-stacks --stack-status-filter UPDATE_COMPLETE | jq -r '.StackSummaries[] | .StackName' | grep "dev-eks-bookstore" | xargs -n1 aws cloudformation delete-stack --stack-name 
```


## Deploying on AWS

EKS cluster expose OIDC service. Following command create IAM identity provider for EKS ODIC service. 
```
eksctl utils associate-iam-oidc-provider  --cluster eks-acg --approve
```

Create IAM policy to attach with k8s service account. Then, run following command 
```
eksctl create iamserviceaccount --name resources-api-iam-serivce-account \
  --namespace development --cluster eks-acg \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/development-ResourcePolicy --approve

# To override existing service account use flag  `--override-existing-serviceaccounts` 
```

Spot Instances 
```
# Create another nodegroup 
eksctl create nodegroup -f cluster.yml

eksctl scale nodegroup --cluster eks-acg --nodes 0 --nodes-min 0 eks-node-group-v2

helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm upgrade --install aws-node-termination-handler -n kube-system --set nodeSelector.lifecycle=Ec2Spot eks/aws-node-termination-handler
```

Farget
```
eksctl create fargetprofile -f cluster.yaml

# to delete all running pod 
kubectl delete pod -n developement `kubectl get po -n development | grep Running | awk '{ print $1 }'`
```

# AppMesh

```
# Disable Custom CNI
kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=false

# Create CRDs for AppMesh controller
kubectl apply -k "https://github.com/aws/eks-charts/stable/appmesh-controller/crds?ref=master"

# Add helm EKS repo
helm repo add eks https://aws.github.io/eks-charts

# Create NameSpace for AppMesh
kubectl create namespace appmesh-system

# Create IAM Role which will be used by AppMesh Controller
eksctl create iamserviceaccount \
  --namespace appmesh-system \
  --name appmesh-controller \
  --attach-policy-arn arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess,arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
  --cluster eks-acg
  --approve

# Install AppMesh Controller
helm upgrade --install appmesh-controller eks/appmesh-controller \
  --namespace appmesh-system \
  --set region=us-east-2 \
  --set serviceAccount.create=false
  --set serviceAccount.name=appmesh-controller \
  --set log.level=debug \
  --set tracing.enable=true --set tracing.provider=x-ray

helm upgrade --install --namespace development clients-api-development . 
```


