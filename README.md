# kubernetes-eks-hands-on
End‑to‑end Kubernetes project on Amazon EKS covering cluster creation with eksctl, IAM OIDC integration, Helm setup, sample app deployment, AWS Load Balancer Controller.
****
# prerequisites
kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.
****
Install Kubectl on Linux machine
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```
Now install eksctl to interact with aws eks
```
# for ARM systems, set ARCH to: `arm64`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
```
****
# Install EKS

Please follow the prerequisites doc before this.

## Install using Fargate

```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```
![img](images/eks-1.png)

![img](images/eks-2.png)

After completing the cluster you can verify the cluster , cloudformation stack, VPC etc on AWS console.
![img](images/eks-4.png)

![img](images/eks-5.png)

![img](images/eks-6.png)

Update Kubeconfig on local server to interact with aws cluster.
```
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```
![img](images/eks-7.png)

Now create one fargate profile 
### 2048 App

## Create Fargate profile
```
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```
![img](images/eks-8.png)

![img](images/eks-9.png)
## Deploy the deployment, service and Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
![img](images/eks-10.png)

Here you will check the created pods , services, ingress

![img](images/eks-11.png)

![img](images/eks-12.png)

![img](images/eks-13.png)

****
# commands to configure IAM OIDC provider 
```
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region us-east-1 \
  --approve
```
![img](images/eks-14.png)

Now ALB controller needs to talk to aws lb so download Iam policy json

# How to setup alb add on

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
![img](images/eks-15.png)

## Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
![img](images/eks-16.png)

## Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
![img](images/eks-17.png)

# Deploy ALB controller

## Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

## Update the repo

```
helm repo update eks
```

## Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>
```
![img](images/eks-18.png)

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![img](images/eks-19.png)

Now check the load balancer was created or not on aws console

![img](images/eks-20.png)

Now check the ingress that will contain address 
```
kubectl get ingress -n game-2048
```
![img](images/eks-21.png)

Now copy paste the address on browser and check whether our 2048-game is coming or not.

![img](images/eks-final-image.png)

after doing all the setup dont forget to delete the cluster.

## Delete the cluster

```
eksctl delete cluster --name demo-cluster --region us-east-1
```









sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
