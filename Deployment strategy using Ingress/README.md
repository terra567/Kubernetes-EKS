# Blue-Green Deployment with AWS ALB Ingress Controller

This project demonstrates implementing blue-green and canary deployments in Amazon EKS using the AWS Application Load Balancer (ALB) Ingress Controller.

## Understanding Deployment Strategies

### Blue-Green Deployment
Blue-green deployment is a technique that reduces downtime by running two identical environments called "Blue" (current version) and "Green" (new version). Traffic is switched from one environment to the other during deployment.

**Benefits:**
- Zero downtime deployments
- Quick rollback capability
- Reduced risk in deployment process
- Testing in production-like environment before switch

### Canary Deployment
Canary deployment involves gradually rolling out changes to a small subset of users before making it available to everyone. Named after the historical practice of using canary birds in coal mines.

**Benefits:**
- Reduced risk by testing with real users
- Gradual rollout of changes
- Early detection of issues
- Ability to test performance with production traffic

## Prerequisites
- Amazon EKS Cluster
- AWS Load Balancer Controller installed (https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/)
- kubectl configured to interact with your cluster

### Commands to deploy resources

- kubectl create -f blueDeploy.yaml
- kubectl create -f blueService.yaml
- kubectl create -f greenDeploy.yaml
- kubectl create -f greenService.yaml
- kubectl create -f ingress.yaml

