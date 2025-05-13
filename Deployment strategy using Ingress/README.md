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

## Commands to deploy resources

- kubectl create -f blueDeploy.yaml
- kubectl create -f blueService.yaml
- kubectl create -f greenDeploy.yaml
- kubectl create -f greenService.yaml
- kubectl create -f ingress.yaml

### Implementation Status
```bash
$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
blue-deployment    2/2     2            2           100m
green-deployment   2/2     2            2           100m

$ kubectl get svc
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
blue-service    ClusterIP   [REDACTED]    <none>        80/TCP    101m
green-service   ClusterIP   [REDACTED]    <none>        80/TCP    101m

$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP           NODE
blue-deployment-978796666-6g7gk     1/1     Running   0          101m   [REDACTED]   ip-192-168-25-96.ec2.internal
blue-deployment-978796666-sbdbc     1/1     Running   0          101m   [REDACTED]   ip-192-168-34-231.ec2.internal
green-deployment-795d58bbd-8lbll    1/1     Running   0          101m   [REDACTED]   ip-192-168-35-111.ec2.internal
green-deployment-795d58bbd-r6h92    1/1     Running   0          101m   [REDACTED]   ip-192-168-16-143.ec2.internal

$ kubectl get ingress
NAME                     CLASS   HOSTS   ADDRESS                                    PORTS   AGE
deploystrategy-ingress   alb     *       [ALB-DNS-NAME]                            80      45m
```

### Traffic Distribution Testing

Testing traffic distribution with 100 requests shows the effectiveness of our weighted routing:

```bash   
$ for i in $(seq 1 100); do 
    curl -s [ALB-DNS-NAME] | awk -F': ' '/^Hostname:/ {print $2}'
done | awk '/blue/ {blue++} /green/ {green++} END {
    print "Blue: " blue " (" (blue/(blue+green)*100) "%)"
    print "Green: " green " (" (green/(blue+green)*100) "%)"
}'

Output:
Blue: 72 (72%)
Green: 28 (28%)
```
Note: The actual traffic distribution might vary slightly from the configured weights (70-30) due to the probabilistic nature of the routing. Larger sample sizes will show distributions closer to the configured weights.

### Traffic Shifting Scenarios
Blue-Green Switch

To switch traffic completely to green:
```bash
alb.ingress.kubernetes.io/actions.weighted-routing: |
  {
    "Type": "forward",
    "ForwardConfig": {
      "TargetGroups": [
        {
          "ServiceName": "blue-service",
          "ServicePort": "80",
          "Weight": 0
        },
        {
          "ServiceName": "green-service",
          "ServicePort": "80",
          "Weight": 100
        }
      ]
    }
  }
```

### Canary Testing

For gradual rollout:

    Start with 90-10
    Move to 70-30
    Then 50-50
    Finally 0-100 if everything is stable

## Clean Up

    
kubectl delete ingress deploystrategy-ingress
kubectl delete svc blue-service green-service
kubectl delete deployment blue-deployment green-deployment

