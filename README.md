# 🚀 Setup Guide

This guide walks through the complete setup process for deploying the **2048 Game** on **Amazon Elastic Kubernetes Service (Amazon EKS)** using **AWS Fargate**.

By the end of this guide, you will have:

- A fully functional Amazon EKS cluster
- AWS Fargate configured as the compute engine
- kubectl connected to your cluster
- AWS Load Balancer Controller installed
- The 2048 Game deployed and publicly accessible through an AWS Application Load Balancer (ALB)

---

# Project Overview

This project uses the following technologies:

| Technology | Purpose |
|------------|---------|
| AWS | Cloud Platform |
| Amazon EKS | Managed Kubernetes Service |
| AWS Fargate | Serverless Compute for Containers |
| Kubernetes | Container Orchestration |
| kubectl | Kubernetes CLI |
| eksctl | EKS Cluster Management |
| AWS CLI | AWS Command Line Interface |
| Helm | Kubernetes Package Manager |
| AWS Load Balancer Controller | Creates Application Load Balancers |

---

# Architecture

```
Internet
     │
     ▼
Application Load Balancer
     │
     ▼
AWS Load Balancer Controller
     │
     ▼
Ingress
     │
     ▼
Service
     │
     ▼
Deployment
     │
     ▼
2048 Game Pod
     │
     ▼
Amazon EKS (Fargate)
```

---

# Prerequisites

Before starting, ensure you have:

- AWS Account
- IAM User (Recommended)
- AdministratorAccess Policy (For Learning Only)
- Internet Connection
- Ubuntu / WSL / Linux
- Basic Kubernetes Knowledge

---

# Step 1 — Create an IAM User

> **Note**
>
> For security reasons, avoid using the AWS Root Account for day-to-day work. Create an IAM User with the required permissions instead.

Navigate to:

```
AWS Console
    ↓
IAM
    ↓
Users
    ↓
Create User
```

### User Configuration

Example:

```
Username

eks-admin
```

Grant the following permission:

```
AdministratorAccess
```

> ⚠️ **For production environments, follow the Principle of Least Privilege instead of using AdministratorAccess.**

📷 **Screenshot**

Add a screenshot after creating the IAM User.

Example:

```
screenshots/
└── Screenshot 2026-07-27 170101.png
```

---

# Step 2 — Create an Access Key

Navigate to:

```
IAM

↓

Users

↓

eks-admin

↓

Security Credentials

↓

Create Access Key
```

Choose:

```
Command Line Interface (CLI)
```

Description Tag (Optional)

Example

```
AWS CLI for EKS Project
```

Click

```
Create Access Key
```

Download or copy:

- Access Key ID
- Secret Access Key

> ⚠️ **Important**
>
> The Secret Access Key is shown **only once**. Store it securely before leaving the page.

📷 **Screenshot**

Add a screenshot of the Access Key creation page (hide sensitive information).

Example:

```
screenshots/
└── Screenshot 2026-07-27 170425.png
```

---

# Step 3 — Install AWS CLI

Verify whether the AWS CLI is already installed.

```bash
aws --version
```

Example Output

```text
aws-cli/2.x.x
```

If it is not installed, follow the official AWS installation guide for your operating system.

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 170810.png
```

---

# Step 4 — Configure AWS CLI

Run

```bash
aws configure
```

Enter

```text
AWS Access Key ID: ********************
AWS Secret Access Key: ********************
Default region name: us-east-1
Default output format: json
```

Verify

```bash
aws sts get-caller-identity
```

Expected Output

```json
{
  "UserId": "XXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/eks-admin"
}
```

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 171034.png
```

---

# Step 5 — Install kubectl

Verify installation

```bash
kubectl version --client
```

Example Output

```text
Client Version: v1.xx.x
```

If kubectl is not installed, install it using the official Kubernetes installation guide.

Verify again

```bash
kubectl version --client
```

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 171208.png
```

---

# Step 6 — Install eksctl

Verify

```bash
eksctl version
```

Expected Output

```text
0.xxx.x
```

If not installed, install it and move the binary into your PATH.

Verify again

```bash
eksctl version
```

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 171412.png
```

---

# Step 7 — Install Helm

Verify

```bash
helm version
```

Expected Output

```text
version.BuildInfo{
...
}
```

If Helm is not installed, install it using the official Helm installation guide.

Verify again

```bash
helm version
```

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 171536.png
```

---

# Step 8 — Verify All Tools

Run the following commands:

```bash
aws --version

kubectl version --client

eksctl version

helm version
```

Expected Result

```text
✓ AWS CLI Installed

✓ kubectl Installed

✓ eksctl Installed

✓ Helm Installed
```

📷 **Screenshot**

```text
screenshots/
└── Screenshot 2026-07-27 171700.png
```

---

# Create and Configure Amazon EKS

In this section, we will create the Amazon EKS cluster and configure all the required AWS components before deploying the application.

By the end of this section, you will have:

- A running Amazon EKS Cluster
- kubectl connected to the cluster
- IAM OIDC Provider configured
- IAM Policy created
- IAM Service Account created

---

# Step 9 — Create the Amazon EKS Cluster

Create the cluster using **eksctl**.

```bash
eksctl create cluster \
--name demo-cluster \
--region us-east-1 \
--fargate
```

The cluster creation process usually takes **15–25 minutes**.

During this process, AWS automatically creates:

- Amazon EKS Cluster
- AWS Fargate Profile
- Amazon VPC
- Public Subnets
- Private Subnets
- Security Groups
- Route Tables
- Internet Gateway
- CloudFormation Stack

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 172105.png
```

---

# Step 10 — Verify the Cluster

List all EKS clusters.

```bash
aws eks list-clusters --region us-east-1
```

Expected Output

```json
{
    "clusters": [
        "demo-cluster"
    ]
}
```

You can also verify using:

```bash
eksctl get cluster
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 172420.png
```

---

# Step 11 — Configure kubectl

Update your kubeconfig so kubectl can communicate with the EKS cluster.

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name demo-cluster
```

Expected Output

```text
Added new context arn:aws:eks:us-east-1:<ACCOUNT_ID>:cluster/demo-cluster to ~/.kube/config
```

Verify the connection.

```bash
kubectl cluster-info
```

Expected Output

```text
Kubernetes control plane is running...
CoreDNS is running...
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 172635.png
```

---

# Step 12 — Verify Cluster Nodes

Since this project uses **AWS Fargate**, there are no EC2 worker nodes.

Run:

```bash
kubectl get nodes
```

Expected Output

```text
No resources found
```

This is expected behavior when using AWS Fargate.

Verify the Fargate profile.

```bash
eksctl get fargateprofile \
--cluster demo-cluster \
--region us-east-1
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 172840.png
```

---

# Step 13 — Associate the IAM OIDC Provider

The AWS Load Balancer Controller requires an IAM OIDC Provider.

Associate it with the cluster.

```bash
eksctl utils associate-iam-oidc-provider \
--cluster demo-cluster \
--region us-east-1 \
--approve
```

Expected Output

```text
IAM Open ID Connect provider is associated with cluster
```

If you receive:

```text
IAM Open ID Connect provider already associated
```

you can safely continue.

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 173050.png
```

---

# Step 14 — Download the IAM Policy

Download the official IAM policy required by the AWS Load Balancer Controller.

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Verify the file exists.

```bash
ls
```

Expected Output

```text
iam_policy.json
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 173235.png
```

---

# Step 15 — Create the IAM Policy

Create the policy in AWS.

```bash
aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```

Expected Output

```text
Policy ARN:
arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy
```

Copy the ARN.

It will be used in the next step.

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 173452.png
```

---

# Step 16 — Create the IAM Service Account

Replace `<ACCOUNT_ID>` with your AWS Account ID.

```bash
eksctl create iamserviceaccount \
--cluster demo-cluster \
--region us-east-1 \
--namespace kube-system \
--name aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--approve
```

This command creates:

- IAM Role
- Kubernetes Service Account
- IAM Role Trust Relationship
- Policy Attachment

---

Verify the Service Account.

```bash
kubectl get serviceaccount \
-n kube-system
```

Expected Output

```text
aws-load-balancer-controller
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 173705.png
```

---

# Step 17 — Verify Everything

Run the following commands.

Verify the cluster.

```bash
kubectl cluster-info
```

Verify namespaces.

```bash
kubectl get ns
```

Verify the service account.

```bash
kubectl get serviceaccount -n kube-system
```

Verify the Fargate profile.

```bash
eksctl get fargateprofile \
--cluster demo-cluster \
--region us-east-1
```

Expected Result

```text
✓ Amazon EKS Cluster Running

✓ kubectl Connected

✓ IAM OIDC Provider Configured

✓ IAM Policy Created

✓ IAM Service Account Created
```

---

📷 **Screenshot**

```
screenshots/
└── Screenshot 2026-07-27 173920.png
```

---

# Deploy the Application

In this section, we will install the AWS Load Balancer Controller and deploy the **2048 Game** on Amazon EKS using AWS Fargate.

By the end of this section, you will have:

- AWS Load Balancer Controller installed
- Kubernetes Namespace created
- Deployment created
- Service created
- Ingress created
- Application Load Balancer (ALB) created
- 2048 Game accessible from the Internet

---

# Step 18 — Add the EKS Helm Repository

Add the official Amazon EKS Helm repository.

```bash
helm repo add eks https://aws.github.io/eks-charts
```

Update the repository.

```bash
helm repo update
```

Verify.

```bash
helm repo list
```

Expected Output

```text
eks    https://aws.github.io/eks-charts
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 174210.png
```

---

# Step 19 — Get the VPC ID

Retrieve the VPC ID associated with your EKS cluster.

```bash
aws eks describe-cluster \
--name demo-cluster \
--region us-east-1 \
--query "cluster.resourcesVpcConfig.vpcId" \
--output text
```

Example Output

```text
vpc-xxxxxxxxxxxxxxxxx
```

Copy the VPC ID.

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 174420.png
```

---

# Step 20 — Install AWS Load Balancer Controller

Replace `<VPC_ID>` with your VPC ID.

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=demo-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller \
--set region=us-east-1 \
--set vpcId=<VPC_ID>
```

Verify the installation.

```bash
helm list -A
```

Expected Output

```text
aws-load-balancer-controller
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 174730.png
```

---

# Step 21 — Verify the Controller

Check whether the controller Pod is running.

```bash
kubectl get pods -n kube-system
```

Expected Output

```text
aws-load-balancer-controller
Running
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 174905.png
```

---

# Step 22 — Create the Namespace

Navigate to the Kubernetes manifests folder.

```bash
cd kubernetes
```

Create the namespace.

```bash
kubectl apply -f namespace.yaml
```

Verify.

```bash
kubectl get ns
```

Expected Output

```text
game-2048
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 175050.png
```

---

# Step 23 — Deploy the Application

Create the Deployment.

```bash
kubectl apply -f deployment.yaml
```

Verify.

```bash
kubectl get deployment -n game-2048
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 175220.png
```

---

# Step 24 — Create the Service

```bash
kubectl apply -f service.yaml
```

Verify.

```bash
kubectl get svc -n game-2048
```

Expected Output

```text
service-2048
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 175355.png
```

---

# Step 25 — Create the Ingress

```bash
kubectl apply -f ingress.yaml
```

Verify.

```bash
kubectl get ingress -n game-2048
```

Initially, the **ADDRESS** column may be empty while AWS provisions the Application Load Balancer.

Wait a few minutes and run the command again.

Example Output

```text
NAME           CLASS   HOSTS   ADDRESS
ingress-2048   alb     *       k8s-game2048-xxxxxxxx.us-east-1.elb.amazonaws.com
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 175640.png
```

---

# Step 26 — Verify the Deployment

Check all resources.

Pods

```bash
kubectl get pods -n game-2048
```

Deployments

```bash
kubectl get deployment -n game-2048
```

Services

```bash
kubectl get svc -n game-2048
```

Ingress

```bash
kubectl get ingress -n game-2048
```

Everything should be in a healthy state.

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 180015.png
```

---

# Step 27 — Access the Application

Copy the **ADDRESS** value from the Ingress.

Example

```text
http://k8s-game2048-xxxxxxxx.us-east-1.elb.amazonaws.com
```

Open the URL in your browser.

If everything has been configured correctly, the **2048 Game** will load successfully.

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 180235.png
```

---

# Step 28 — Validate the Deployment

Run the following commands.

Check Pods

```bash
kubectl get pods -n game-2048
```

Check Services

```bash
kubectl get svc -n game-2048
```

Check Ingress

```bash
kubectl get ingress -n game-2048
```

Check Helm

```bash
helm list -A
```

Check Cluster

```bash
kubectl cluster-info
```

Expected Result

```text
✓ Amazon EKS Cluster Running

✓ AWS Load Balancer Controller Running

✓ Namespace Created

✓ Deployment Running

✓ Pods Running

✓ Service Running

✓ Ingress Running

✓ Application Load Balancer Created

✓ 2048 Game Accessible
```

---

📷 Screenshot

```
screenshots/
└── Screenshot 2026-07-27 180520.png
```

---

# Congratulations 🎉

You have successfully deployed the **2048 Game** on **Amazon EKS** using **AWS Fargate**.

During this project, you learned how to:

- Create an Amazon EKS Cluster
- Configure AWS CLI and kubectl
- Install and use eksctl
- Configure IAM OIDC Provider
- Create IAM Policies and Service Accounts
- Install the AWS Load Balancer Controller with Helm
- Deploy a containerized application to Kubernetes
- Expose an application using an AWS Application Load Balancer
- Verify and troubleshoot Kubernetes resources

This project demonstrates practical experience with **AWS**, **Kubernetes**, **Amazon EKS**, **AWS Fargate**, **IAM**, **Helm**, **Ingress**, and the **AWS Load Balancer Controller**, making it an excellent portfolio project for DevOps and Cloud Engineering roles.

1 2 3 4 5 CLEA 6 7 8 9 10 