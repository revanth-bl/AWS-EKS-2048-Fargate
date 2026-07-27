# 🏗️ Architecture

This document explains the architecture used in this project and how every AWS and Kubernetes component works together to expose the application to the Internet.

---

# Architecture Diagram

```text
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
                    2048 Game Application
                               │
                               ▼
                      AWS Fargate Pod
                               │
                               ▼
                      Amazon EKS Cluster
```

---

# Architecture Overview

This project uses **Amazon Elastic Kubernetes Service (Amazon EKS)** with **AWS Fargate** to deploy a Kubernetes application without managing EC2 worker nodes.

The application is exposed to the Internet through an **Application Load Balancer (ALB)** created automatically by the **AWS Load Balancer Controller**.

---

# Request Flow

When a user opens the application URL, the request follows this path:

```
User

↓

Application Load Balancer (ALB)

↓

AWS Load Balancer Controller

↓

Ingress

↓

Kubernetes Service

↓

Deployment

↓

Pod

↓

2048 Game
```

---

# Component Breakdown

---

## 1. Amazon EKS

Amazon Elastic Kubernetes Service (Amazon EKS) is a managed Kubernetes service provided by AWS.

Instead of installing and maintaining Kubernetes manually, AWS manages the Kubernetes Control Plane.

Responsibilities:

- Kubernetes API Server
- Scheduler
- etcd Database
- High Availability
- Cluster Management

Advantages

- Fully Managed
- Highly Available
- Secure
- Integrated with AWS Services

---

## 2. AWS Fargate

AWS Fargate is a serverless compute engine for containers.

Unlike traditional Kubernetes clusters, there are **no EC2 worker nodes**.

When a Pod is created, AWS automatically provisions the compute resources needed to run it.

Advantages

- No EC2 management
- Automatic scaling
- Improved security
- Pay only for resources consumed

---

## 3. Kubernetes Deployment

The Deployment object is responsible for managing application Pods.

Responsibilities

- Creates Pods
- Maintains desired number of replicas
- Self-healing
- Rolling Updates
- Rollbacks

Example

```
Deployment

↓

ReplicaSet

↓

Pod
```

---

## 4. Pod

A Pod is the smallest deployable unit in Kubernetes.

In this project the Pod runs the **2048 Game container**.

Each Pod receives

- IP Address
- Storage
- Network
- Container Runtime

---

## 5. Kubernetes Service

Pods are temporary.

Whenever Pods restart, their IP addresses change.

A Kubernetes Service provides a stable network endpoint.

Responsibilities

- Stable IP
- Service Discovery
- Load Balancing
- Internal Communication

Without a Service, the Ingress would not know where to forward traffic.

---

## 6. Kubernetes Ingress

Ingress manages HTTP and HTTPS traffic entering the Kubernetes cluster.

Instead of exposing each Service individually, Ingress provides centralized routing.

Example

```
/

↓

2048 Service
```

An Ingress does **not** create an AWS Load Balancer by itself.

That responsibility belongs to the AWS Load Balancer Controller.

---

## 7. AWS Load Balancer Controller

The AWS Load Balancer Controller watches Kubernetes resources.

Whenever an Ingress resource is created, it automatically creates an AWS Application Load Balancer.

Responsibilities

- Creates ALB
- Creates Target Groups
- Registers Pods
- Updates Routing Rules
- Deletes Resources when removed

Without this controller, Kubernetes cannot automatically provision AWS Load Balancers.

---

## 8. Application Load Balancer (ALB)

The ALB is the public entry point to the application.

Users connect to the ALB instead of directly connecting to Kubernetes.

Responsibilities

- HTTP Routing
- Health Checks
- SSL/TLS Support
- Load Balancing
- High Availability

The ALB is automatically created after the Ingress is applied.

---

## 9. IAM OIDC Provider

The AWS Load Balancer Controller requires permission to create AWS resources.

Instead of storing AWS credentials inside Kubernetes, this project uses **IAM Roles for Service Accounts (IRSA)**.

The OIDC Provider allows Kubernetes Service Accounts to securely assume IAM Roles.

Benefits

- No Access Keys
- Temporary Credentials
- Better Security
- AWS Best Practice

---

## 10. IAM Service Account

The controller runs inside Kubernetes.

It needs AWS permissions.

Instead of giving permissions to every Pod, only the **aws-load-balancer-controller Service Account** receives the required IAM Role.

This follows the Principle of Least Privilege.

---

# Why Fargate Instead of EC2?

Traditional Kubernetes

```
EC2

↓

Kubernetes Worker Nodes

↓

Pods
```

This Project

```
AWS Fargate

↓

Pods
```

Benefits

- No EC2 Instances
- No Auto Scaling Groups
- No Node Maintenance
- Less Operational Overhead

---

# Resources Created During This Project

AWS Resources

- Amazon EKS Cluster
- Amazon VPC
- Subnets
- Security Groups
- IAM Roles
- IAM Policies
- OIDC Provider
- Application Load Balancer
- Target Groups
- CloudFormation Stack

Kubernetes Resources

- Namespace
- Deployment
- Pod
- Service
- Ingress
- Service Account

---

# Security Considerations

This project follows several AWS security best practices.

- IAM User instead of Root Account
- IAM Roles for Service Accounts (IRSA)
- OIDC Authentication
- Temporary AWS Credentials
- Least Privilege Access

For production workloads, avoid using the `AdministratorAccess` policy. Instead, grant only the permissions required for the specific application.

---

# Key Learning Outcomes

After completing this project you should understand:

- Amazon EKS Architecture
- Kubernetes Components
- AWS Fargate
- IAM Roles for Service Accounts
- OIDC Authentication
- Kubernetes Networking
- Application Load Balancer
- AWS Load Balancer Controller
- Kubernetes Deployments
- Kubernetes Services
- Kubernetes Ingress

---

# Conclusion

This project demonstrates how a containerized application can be deployed on a managed Kubernetes platform using AWS-native services.

By combining **Amazon EKS**, **AWS Fargate**, **AWS Load Balancer Controller**, **Ingress**, and an **Application Load Balancer**, we can expose applications securely to the Internet without managing Kubernetes worker nodes.

This architecture is scalable, secure, and aligns with modern cloud-native deployment practices.