# 🧹 Cleanup Guide

## Overview

Amazon EKS is **not included in the AWS Free Tier**. Running an EKS cluster, Application Load Balancer (ALB), and related networking resources can incur charges.

After completing this project, it is recommended to delete all resources to avoid unnecessary costs.

This guide walks through the complete cleanup process.

---

# Resources Created

During this project, the following resources were created.

### AWS Resources

- Amazon EKS Cluster
- AWS Fargate Profile
- Amazon VPC
- Public Subnets
- Private Subnets
- Security Groups
- Internet Gateway
- Route Tables
- Application Load Balancer (ALB)
- Target Groups
- IAM Role
- IAM Policy
- IAM OIDC Provider
- CloudFormation Stack

### Kubernetes Resources

- Namespace
- Deployment
- Pod
- Service
- Ingress
- Service Account

---

# Cleanup Order

Delete resources in the following order.

```
Application

↓

AWS Load Balancer Controller

↓

IAM Service Account

↓

Amazon EKS Cluster

↓

IAM Role

↓

IAM Policy

↓

OIDC Provider

↓

Verify AWS Resources
```

---

# Step 1 — Delete the Application (Optional)

Delete the Kubernetes application resources.

```bash
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete -f namespace.yaml
```

Verify

```bash
kubectl get all -A
```

---

# Step 2 — Uninstall AWS Load Balancer Controller

```bash
helm uninstall aws-load-balancer-controller \
-n kube-system
```

Verify

```bash
helm list -A
```

Expected Output

```
No releases found.
```

---

# Step 3 — Delete IAM Service Account

```bash
eksctl delete iamserviceaccount \
--cluster demo-cluster \
--region us-east-1 \
--namespace kube-system \
--name aws-load-balancer-controller
```

Verify

```bash
kubectl get serviceaccount \
-n kube-system
```

The service account should no longer exist.

---

# Step 4 — Delete the Amazon EKS Cluster

Run

```bash
eksctl delete cluster \
--name demo-cluster \
--region us-east-1
```

This process usually takes **10–20 minutes**.

The following resources will be deleted automatically:

- Amazon EKS Cluster
- Fargate Profile
- CloudFormation Stack
- VPC (if created by eksctl)
- Subnets
- Security Groups
- Route Tables
- Internet Gateway

---

Verify

```bash
aws eks list-clusters \
--region us-east-1
```

Expected Output

```json
{
    "clusters": []
}
```

---

# Step 5 — Delete the IAM Policy

Navigate to

```
AWS Console

↓

IAM

↓

Policies

↓

AWSLoadBalancerControllerIAMPolicy

↓

Delete
```

---

# Step 6 — Delete the IAM Role

Navigate to

```
AWS Console

↓

IAM

↓

Roles

↓

AmazonEKSLoadBalancerControllerRole

↓

Delete
```

---

# Step 7 — Delete the IAM OIDC Provider

Navigate to

```
AWS Console

↓

IAM

↓

Identity Providers

↓

Delete the OIDC Provider associated with demo-cluster
```

---

# Step 8 — Verify Application Load Balancer

Navigate to

```
AWS Console

↓

EC2

↓

Load Balancers
```

There should be **no Application Load Balancer** associated with this project.

If one still exists after several minutes, verify that the Ingress resource has been deleted.

---

# Step 9 — Verify Target Groups

Navigate to

```
AWS Console

↓

EC2

↓

Target Groups
```

There should be no target groups related to this project.

---

# Step 10 — Verify CloudFormation

Navigate to

```
AWS Console

↓

CloudFormation

↓

Stacks
```

There should be no stacks beginning with:

```
eksctl-demo-cluster
```

---

# Step 11 — Verify VPC

Navigate to

```
AWS Console

↓

VPC

↓

Your VPCs
```

If the VPC was created automatically by `eksctl`, it should already be deleted.

---

# Step 12 — Final Verification

Run the following commands.

Verify EKS

```bash
aws eks list-clusters \
--region us-east-1
```

Verify Kubernetes Context

```bash
kubectl config get-contexts
```

Verify Helm

```bash
helm list -A
```

Expected Result

- No EKS Cluster
- No AWS Load Balancer Controller
- No Application Load Balancer
- No Target Groups
- No CloudFormation Stack
- No IAM Role (created for this project)
- No IAM Policy (created for this project)
- No OIDC Provider (created for this project)

---

# Troubleshooting

## Cluster deletion is taking a long time

This is normal.

Deleting an Amazon EKS cluster may take between **10–20 minutes**.

---

## Load Balancer still exists

Ensure that:

- The Ingress resource has been deleted.
- The AWS Load Balancer Controller was uninstalled.
- The EKS cluster deletion has completed.

---

## IAM Policy cannot be deleted

Verify that no IAM Roles are attached to the policy.

Delete the IAM Role first.

---

## OIDC Provider cannot be deleted

Ensure that all IAM Roles using the OIDC Provider have been removed.

---

# Best Practices

- Always delete cloud resources after completing a learning project.
- Use an IAM User instead of the AWS Root Account.
- Regularly review the AWS Billing Dashboard.
- Monitor active resources to avoid unexpected charges.
- Apply the Principle of Least Privilege for production environments.

---

# Conclusion

You have successfully removed all resources created during this project.

Your AWS account should no longer contain any infrastructure related to the EKS 2048 Game deployment, helping prevent unnecessary costs while keeping your environment clean and ready for future projects.