# Game 2048 Using EKS and Ingress (AWS loadbalancer controller)

## Install EKS

### Install using Fargate

```bash
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

## commands to configure IAM OIDC provider

Run the below command

```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

## How to setup alb controller add on (Refer to link: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/deploy/installation/) 

### Download IAM policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

### Create IAM Policy

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

### Create IAM Role

```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Deploy ALB controller

### Add helm repo

```bash
helm repo add eks https://aws.github.io/eks-charts
```

### Update the repo

```bash
helm repo update eks
```

### Install AWS Load Balancer controller
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```

### Verify that the deployments are running.

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

## 2048 App
### Create Fargate profile

```bash
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

### Deploy the deployment, service and Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```


![image](https://github.com/user-attachments/assets/fd422e2a-c616-4b10-b20d-63e99778a275)

