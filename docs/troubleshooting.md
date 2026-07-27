# 🛠️ Troubleshooting Guide

This document contains the most common issues you may encounter while deploying the **2048 Game on Amazon EKS using AWS Fargate**, along with their solutions.

Most of these issues were encountered during the development of this project and are included here to help others troubleshoot quickly.

---

# Table of Contents

- AWS CLI Issues
- eksctl Issues
- kubectl Issues
- EKS Cluster Issues
- OIDC Issues
- IAM Issues
- Helm Issues
- AWS Load Balancer Controller Issues
- Kubernetes Issues
- Ingress Issues
- Application Issues

---

# AWS CLI Issues

## AWS Region must be set

### Error

```text
Error:
AWS Region must be set,
please set the AWS Region in AWS config file
or as environment variable.
```

### Cause

AWS CLI has not been configured with a default region.

### Solution

Run

```bash
aws configure
```

or

```bash
aws configure set region us-east-1
```

Verify

```bash
aws configure list
```

---

## Invalid Access Key

### Error

```text
InvalidClientTokenId
```

### Cause

Incorrect AWS Access Key or Secret Access Key.

### Solution

Run

```bash
aws configure
```

Enter the correct credentials.

Verify

```bash
aws sts get-caller-identity
```

---

# eksctl Issues

## eksctl command not found

### Error

```text
eksctl: command not found
```

### Cause

eksctl is not installed or is not in your PATH.

### Solution

Verify

```bash
eksctl version
```

If not installed, download and install it again.

---

## tar cannot open file

### Error

```text
tar:
Cannot open:
No such file or directory
```

### Cause

The eksctl archive was not downloaded successfully.

### Solution

Verify the file exists

```bash
ls
```

Download again

```bash
curl -LO https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz
```

Extract

```bash
tar -xzf eksctl_Linux_amd64.tar.gz
```

Move

```bash
sudo mv eksctl /usr/local/bin/
```

Verify

```bash
eksctl version
```

---

# kubectl Issues

## kubectl cannot connect

### Error

```text
Unable to connect to the server
```

### Cause

kubeconfig is missing or incorrect.

### Solution

Run

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name demo-cluster
```

Verify

```bash
kubectl cluster-info
```

---

# EKS Cluster Issues

## Cluster creation takes a long time

### Cause

Amazon EKS typically takes between 15–25 minutes to provision.

### Solution

Wait until CloudFormation completes.

Check progress

```bash
eksctl get cluster
```

---

## Cluster not found

### Error

```text
No cluster found
```

### Solution

Verify

```bash
aws eks list-clusters
```

Ensure you are using the correct AWS region.

---

# OIDC Issues

## OIDC Provider already exists

### Output

```text
IAM Open ID Connect provider already associated
```

### Explanation

This is **not an error**.

The cluster already has an associated OIDC provider.

You can continue with the next step.

---

# IAM Issues

## AccessDenied

### Error

```text
AccessDeniedException
```

### Cause

The IAM User lacks required permissions.

### Solution

For learning purposes, attach

```
AdministratorAccess
```

to your IAM User.

---

## Unable to create IAM Service Account

### Cause

Most commonly:

- Incorrect Account ID
- IAM Policy does not exist
- OIDC Provider not associated

### Verify

```bash
aws iam list-policies
```

---

# Helm Issues

## helm command not found

### Cause

Helm is not installed.

### Verify

```bash
helm version
```

Install Helm if necessary.

---

## Repository not found

### Solution

Run

```bash
helm repo add eks https://aws.github.io/eks-charts
```

Update

```bash
helm repo update
```

---

# AWS Load Balancer Controller Issues

## Controller Pod not running

Verify

```bash
kubectl get pods -n kube-system
```

If the controller is not running:

- Verify IAM Role
- Verify Service Account
- Verify Helm installation

---

## Helm installation failed

Verify

```bash
helm list -A
```

If missing

Reinstall the controller.

---

# Kubernetes Issues

## Pods stuck in Pending

Possible causes

- Missing Fargate Profile
- Incorrect Namespace
- Deployment error

Verify

```bash
kubectl describe pod <pod-name> -n game-2048
```

---

## Pods in CrashLoopBackOff

Check logs

```bash
kubectl logs <pod-name> -n game-2048
```

Describe Pod

```bash
kubectl describe pod <pod-name> -n game-2048
```

---

# Ingress Issues

## ADDRESS remains empty

### Cause

The Application Load Balancer is still being provisioned.

### Solution

Wait several minutes.

Verify

```bash
kubectl get ingress -n game-2048
```

---

## ALB URL not accessible

Verify

```bash
kubectl get pods -n game-2048
```

```bash
kubectl get svc -n game-2048
```

```bash
kubectl get ingress -n game-2048
```

Ensure every resource is in the **Running** state.

---

# Application Issues

## 2048 Game does not load

Verify

- Deployment exists
- Pod is Running
- Service exists
- Ingress exists
- ALB has been created

Restart the Deployment

```bash
kubectl rollout restart deployment deployment-2048 -n game-2048
```

---

# Useful Verification Commands

Check Cluster

```bash
kubectl cluster-info
```

Check Namespaces

```bash
kubectl get ns
```

Check Pods

```bash
kubectl get pods -A
```

Check Services

```bash
kubectl get svc -A
```

Check Ingress

```bash
kubectl get ingress -A
```

Check Deployments

```bash
kubectl get deployment -A
```

Check Helm

```bash
helm list -A
```

Check AWS Identity

```bash
aws sts get-caller-identity
```

Check EKS Cluster

```bash
aws eks list-clusters --region us-east-1
```

---

# Best Practices

- Always use an IAM User instead of the Root Account.
- Configure a default AWS Region before using `eksctl`.
- Verify each step before moving to the next.
- Wait for AWS resources to finish provisioning before troubleshooting.
- Clean up resources after completing the project to avoid unnecessary charges.

---

# Conclusion

Troubleshooting is a normal part of working with Kubernetes and AWS. Understanding the cause of an issue is often more valuable than memorizing commands.

The issues documented here reflect real-world scenarios encountered during this project and provide practical solutions for resolving them efficiently.