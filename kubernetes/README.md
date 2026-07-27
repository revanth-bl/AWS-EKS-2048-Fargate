# ☸️ Kubernetes Manifests

This directory contains all the Kubernetes resource definitions required to deploy the **2048 Game** on **Amazon EKS** using **AWS Fargate**.

Each manifest has a specific responsibility in the deployment process.

---

# Folder Structure

```
kubernetes/
├── namespace.yaml
├── deployment.yaml
├── service.yaml
└── ingress.yaml
```

---

# Deployment Flow

The resources should be applied in the following order.

```
Namespace
    │
    ▼
Deployment
    │
    ▼
Service
    │
    ▼
Ingress
```

Commands

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

# Resource Overview

| File | Purpose |
|-------|---------|
| namespace.yaml | Creates the Kubernetes Namespace |
| deployment.yaml | Deploys the 2048 Game Pod |
| service.yaml | Exposes the Pod internally |
| ingress.yaml | Creates the public Application Load Balancer |

---

# namespace.yaml

Creates a dedicated namespace named **game-2048**.

Keeping resources inside their own namespace improves organization and simplifies management.

Apply

```bash
kubectl apply -f namespace.yaml
```

Verify

```bash
kubectl get ns
```

Expected Output

```
game-2048
```

---

# deployment.yaml

Creates the Deployment responsible for running the 2048 Game container.

Responsibilities

- Creates Pods
- Maintains desired replicas
- Restarts failed Pods
- Supports rolling updates

Apply

```bash
kubectl apply -f deployment.yaml
```

Verify

```bash
kubectl get deployment -n game-2048
```

Verify Pods

```bash
kubectl get pods -n game-2048
```

Expected Output

```
deployment-2048
```

Pod Status

```
Running
```

---

# service.yaml

Creates a Kubernetes Service.

The Service provides a stable endpoint for the Pods.

Without a Service, the Ingress cannot forward traffic.

Apply

```bash
kubectl apply -f service.yaml
```

Verify

```bash
kubectl get svc -n game-2048
```

Expected Output

```
service-2048
```

---

# ingress.yaml

Creates a Kubernetes Ingress resource.

The AWS Load Balancer Controller detects this resource and automatically provisions an **Application Load Balancer (ALB)**.

Apply

```bash
kubectl apply -f ingress.yaml
```

Verify

```bash
kubectl get ingress -n game-2048
```

Expected Output

```
ingress-2048
```

The ADDRESS column should display the ALB DNS name after a few minutes.

Example

```
k8s-game2048-xxxxxxxx.us-east-1.elb.amazonaws.com
```

---

# Verify All Resources

Deployment

```bash
kubectl get deployment -n game-2048
```

Pods

```bash
kubectl get pods -n game-2048
```

Service

```bash
kubectl get svc -n game-2048
```

Ingress

```bash
kubectl get ingress -n game-2048
```

---

# Common Issues

## Pod is Pending

Possible causes

- Fargate profile is missing
- Deployment configuration is incorrect

---

## Service Not Found

Verify that the Deployment and Pods are running before creating the Service.

---

## Ingress ADDRESS is Empty

Wait several minutes for the AWS Load Balancer Controller to provision the ALB.

Verify the controller is running.

```bash
kubectl get pods -n kube-system
```

---

## ALB URL Does Not Load

Verify

- Pods are Running
- Service exists
- Ingress exists
- AWS Load Balancer Controller is Running

---

# Learning Outcomes

By understanding these manifests, you will learn:

- Kubernetes Namespaces
- Deployments
- Pods
- Services
- Ingress
- AWS Load Balancer Controller
- Application Load Balancer (ALB)
- Traffic Flow in Kubernetes

---

# Next Step

After applying all manifests successfully, open the ALB URL displayed by:

```bash
kubectl get ingress -n game-2048
```

If the setup is correct, the **2048 Game** will be accessible from your browser through the AWS Application Load Balancer.