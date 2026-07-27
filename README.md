# 🚀 Deploying the 2048 Game on Amazon EKS using AWS Fargate

<p align="center">

![AWS](https://img.shields.io/badge/AWS-EKS-orange?style=for-the-badge&logo=amazonaws)
![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.33-blue?style=for-the-badge&logo=kubernetes)
![Helm](https://img.shields.io/badge/Helm-v3-0F1689?style=for-the-badge&logo=helm)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Project-Completed-success?style=for-the-badge)

</p>

---

## 📖 Project Overview

This project demonstrates how to deploy a Kubernetes application on **Amazon Elastic Kubernetes Service (Amazon EKS)** using **AWS Fargate** as the compute engine.

Instead of managing EC2 worker nodes, the application runs on **AWS Fargate**, allowing Kubernetes Pods to execute in a fully managed serverless environment.

The application is exposed to the internet using:

- AWS Load Balancer Controller
- Kubernetes Ingress
- Application Load Balancer (ALB)

The sample application used throughout this project is the classic **2048 Game**.

---

## 🎯 Project Objectives

By completing this project you will learn how to:

✅ Create an Amazon EKS Cluster  
✅ Configure kubectl  
✅ Use eksctl to manage Kubernetes  
✅ Configure IAM Roles for Service Accounts (IRSA)  
✅ Associate IAM OIDC Provider  
✅ Install AWS Load Balancer Controller  
✅ Deploy applications on Kubernetes  
✅ Create Kubernetes Deployment, Service, and Ingress  
✅ Automatically provision an AWS Application Load Balancer  
✅ Expose Kubernetes applications to the Internet  

---

## 🏗️ Architecture

```
                       Internet
                           │
                           ▼
             AWS Application Load Balancer
                           │
                           ▼
           AWS Load Balancer Controller
                           │
                           ▼
                Kubernetes Ingress
                           │
                           ▼
                 Kubernetes Service
                           │
                           ▼
              Kubernetes Deployment
                           │
                           ▼
              2048 Game Pod (Fargate)
```

---

## ⚙️ Technologies Used

| Technology | Purpose |
|------------|---------|
| Amazon EKS | Managed Kubernetes |
| AWS Fargate | Serverless Kubernetes Compute |
| Kubernetes | Container Orchestration |
| AWS CLI | AWS Management |
| kubectl | Kubernetes CLI |
| eksctl | EKS Management |
| Helm | Kubernetes Package Manager |
| IAM | Identity & Access Management |
| OIDC | Secure IAM Authentication |
| Application Load Balancer | Internet Traffic Routing |

---

## ☁️ AWS Services Used

- Amazon EKS
- AWS Fargate
- IAM + IAM OIDC Provider
- Application Load Balancer
- Amazon VPC
- Elastic Load Balancing
- CloudFormation
- EC2 Networking Components

---

## 📂 Repository Structure

```
aws-eks-2048-fargate
│
├── README.md
│
├── kubernetes
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
├── screenshots
├── assets
└── docs
    ├── setup.md
    ├── architecture.md
    ├── troubleshooting.md
    ├── cleanup.md
    └── interview-questions.md
```

---

## 📋 Prerequisites

Before beginning this project ensure you have:

- AWS Account with IAM User
- AdministratorAccess Policy (Learning Environment)
- AWS CLI, kubectl, eksctl, and Helm installed
- Internet Connection

---

## 💰 Cost Warning

> ⚠️ **Important:** Amazon EKS is **not included in the AWS Free Tier**. Running an EKS cluster incurs charges.

Always delete the following after completing the project:

- EKS Cluster
- Application Load Balancer + Target Groups
- IAM Roles (if created only for this project)
- OIDC Provider
- CloudFormation Stack

---

## 👤 Step 1 — Create an IAM User

Instead of using the AWS Root Account, create a dedicated IAM User.

Navigate to: **AWS Console → IAM → Users → Create User**

**User Name:** `revan-devops`  
**Permissions:** Attach `AdministratorAccess`

Then generate an Access Key:

**IAM User → Security Credentials → Create Access Key → Command Line Interface (CLI)**

Download the CSV or copy your **Access Key ID** and **Secret Access Key**.

---

## 💻 Step 2 — Install & Verify AWS CLI

```bash
aws --version
# aws-cli/2.xx.x
```

---

## ☸️ Step 3 — Install & Verify kubectl

```bash
kubectl version --client
# Client Version: v1.xx.x
```

---

## ⚡ Step 4 — Install & Verify eksctl

```bash
eksctl version
# 0.xxx.x
```

---

## ⛵ Step 5 — Install & Verify Helm

```bash
helm version
# version.BuildInfo
```

---

## 🔑 Step 6 — Configure AWS CLI

```bash
aws configure
# AWS Access Key ID: <your-key>
# AWS Secret Access Key: <your-secret>
# Default region: us-east-1
# Default output format: json
```

Verify your identity:

```bash
aws sts get-caller-identity
```

Expected Output:

```json
{
  "UserId": "...",
  "Account": "...",
  "Arn": "arn:aws:iam::<ACCOUNT_ID>:user/revan-devops"
}
```

---

## 🚀 Step 7 — Create the Amazon EKS Cluster

```bash
eksctl create cluster \
  --name demo-cluster \
  --region us-east-1 \
  --fargate
```

This single command automatically provisions: EKS Cluster, VPC, Subnets, Security Groups, IAM Roles, CloudFormation Stack, and a Fargate Profile.  
**Takes 15–25 minutes.**

Verify:

```bash
aws eks list-clusters --region us-east-1
kubectl cluster-info
```

---

## ⚙️ Step 8 — Update kubeconfig

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name demo-cluster
```

Verify:

```bash
kubectl config current-context
# arn:aws:eks:us-east-1:xxxxxxxx:cluster/demo-cluster
```

---

## 🔍 Step 9 — Verify the Cluster

```bash
kubectl get nodes    # No resources found — expected with Fargate
kubectl get ns       # default, kube-system, kube-public, kube-node-lease
kubectl get pods -A
```

> ℹ️ No EC2 worker nodes appear with Fargate — this is completely normal.

---

## 🔐 Step 10 — Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region us-east-1 \
  --approve
```

---

## 🛡️ Step 11 — Create IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

## 👤 Step 12 — Create IAM Service Account

Replace `<ACCOUNT_ID>` with your AWS Account ID:

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --region=us-east-1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name=AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

Verify:

```bash
kubectl get serviceaccount -n kube-system
```

---

## ⛵ Step 13 — Install AWS Load Balancer Controller

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Get your VPC ID:

```bash
aws eks describe-cluster \
  --name demo-cluster \
  --region us-east-1 \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text
```

Install:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<YOUR_VPC_ID>
```

Verify:

```bash
helm list -A
kubectl get deployment -n kube-system
kubectl get pods -n kube-system
```

The controller Pod should be in the **Running** state.

---

## 🎮 Step 14 — Deploy the 2048 Application

```bash
kubectl apply -f kubernetes/namespace.yaml
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/ingress.yaml
```

---

## ✅ Step 15 — Verify Kubernetes Resources

```bash
kubectl get deployment -n game-2048
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
```

Expected ingress output:

```
NAME           CLASS   HOSTS   ADDRESS
ingress-2048   alb     *       k8s-game2048-xxxxxxxx.us-east-1.elb.amazonaws.com
```

---

## 🌍 Step 16 — Access the Application

Copy the **ADDRESS** from the ingress output and open it in your browser:

```
http://k8s-game2048-xxxxxxxx.us-east-1.elb.amazonaws.com
```

If the **2048 Game** loads — you've successfully deployed a containerized application on Amazon EKS with AWS Fargate! 🎉

**Request flow:**

```
User → ALB → Load Balancer Controller → Ingress → Service → Deployment → Pod → Fargate
```

---

## 📊 Final Verification Checklist

```bash
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
kubectl get deployment -n game-2048
helm list -A
aws eks list-clusters --region us-east-1
```

---

## 🧹 Cleanup

```bash
# Delete Load Balancer Controller
helm uninstall aws-load-balancer-controller -n kube-system

# Delete IAM Service Account
eksctl delete iamserviceaccount \
  --cluster demo-cluster \
  --region us-east-1 \
  --namespace kube-system \
  --name aws-load-balancer-controller

# Delete EKS Cluster
eksctl delete cluster \
  --name demo-cluster \
  --region us-east-1
```

Verify deletion:

```bash
aws eks list-clusters --region us-east-1
# { "clusters": [] }
```

---

## 🛠️ Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| AWS Region must be set | `aws configure set region us-east-1` |
| `AccessDeniedException` | Verify your IAM User has the required permissions |
| `No resources found` on `kubectl get nodes` | Expected with Fargate — no EC2 nodes are created |
| Ingress ADDRESS is empty | Wait a few minutes for ALB provisioning to complete |
| ALB URL does not load | Check controller is running, ingress exists, pods are Running |

---

## 💡 Key Concepts Learned

- Amazon EKS + AWS Fargate
- kubectl, eksctl, Helm
- IAM Roles for Service Accounts (IRSA)
- OIDC Provider
- Application Load Balancer
- Kubernetes Deployment, Service, and Ingress

---

## 📚 References

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks)
- [Kubernetes Documentation](https://kubernetes.io/docs)
- [eksctl Documentation](https://eksctl.io)
- [Helm Documentation](https://helm.sh/docs)
- [AWS Load Balancer Controller Documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller)

---

## 🙏 Acknowledgements

This project was completed by following and expanding upon the excellent **AWS DevOps Zero to Hero** series by **Abhishek Veeramalla**. Enhanced with additional explanations, verification steps, troubleshooting tips, and documentation to serve as a complete learning resource.

---

## ⭐ Support

If you found this repository helpful — **star it**, fork it, and try deploying your own app on EKS!

Happy Learning! 🚀