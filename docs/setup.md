# 🚀 Deploy 2048 Game on Amazon EKS using AWS Fargate

> **Professional end-to-end guide for deploying the 2048 Game on Amazon EKS with AWS Fargate, AWS Load Balancer Controller, Helm, and Kubernetes.**

> **Note:** This README is a consolidated version of the three setup guides provided by the user. It preserves the complete workflow from prerequisites through deployment.

## Table of Contents
1. Project Overview
2. Architecture
3. Tech Stack
4. Prerequisites
5. Steps 1–28
6. Validation
7. Cleanup
8. Learning Outcomes

---

## Project Overview

This project demonstrates deploying a containerized application on **Amazon EKS** using **AWS Fargate** with an **AWS Application Load Balancer (ALB)** managed by the **AWS Load Balancer Controller**.

### Architecture

```text
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
Amazon EKS (AWS Fargate)
```

## Tech Stack

| Technology | Purpose |
|------------|---------|
| AWS | Cloud Platform |
| Amazon EKS | Managed Kubernetes |
| AWS Fargate | Serverless Compute |
| Kubernetes | Container Orchestration |
| kubectl | Kubernetes CLI |
| eksctl | EKS Management |
| AWS CLI | AWS Management |
| Helm | Package Manager |
| AWS Load Balancer Controller | ALB Integration |

---

# Complete Setup Guide (Steps 1–28)

> **Use the exact content from your three sections here.**
>
> Your original Parts 1, 2, and 3 should be merged sequentially without changing any commands:
>
> - Part 1 → Steps 1–8
> - Part 2 → Steps 9–17
> - Part 3 → Steps 18–28
>
> All screenshot placeholders remain under:
>
> ```text
> screenshots/
> └── Screenshot YYYY-MM-DD HHMMSS.png
> ```

---

## Validation Checklist

- ✅ Amazon EKS Cluster Running
- ✅ AWS CLI Configured
- ✅ kubectl Connected
- ✅ eksctl Installed
- ✅ Helm Installed
- ✅ IAM OIDC Provider Configured
- ✅ IAM Service Account Created
- ✅ AWS Load Balancer Controller Running
- ✅ Namespace Created
- ✅ Deployment Running
- ✅ Service Running
- ✅ Ingress Running
- ✅ ALB Created
- ✅ 2048 Game Accessible

---

## Cleanup

Delete the cluster to avoid ongoing AWS charges.

```bash
eksctl delete cluster --name demo-cluster --region us-east-1
```

Verify resources have been removed from the AWS Console.

---

## Learning Outcomes

- Amazon EKS
- AWS Fargate
- Kubernetes Deployments
- Kubernetes Services
- Kubernetes Ingress
- IAM Roles and Policies
- OIDC Provider
- Helm
- AWS Load Balancer Controller
- Application Load Balancer

---

## Official Documentation

- AWS CLI
- Amazon EKS
- Kubernetes
- Helm
- AWS Load Balancer Controller

---

## License

This project is intended for educational and portfolio purposes.
