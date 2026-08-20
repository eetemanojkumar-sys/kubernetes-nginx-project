# Production-Oriented Kubernetes NGINX Deployment

## Overview

A hands-on Kubernetes project that goes beyond a basic Pod deployment and demonstrates several production-oriented concepts around application delivery, networking, configuration, storage, scaling, health checks, and access control.

The project was built and tested with Minikube running on an AWS EC2 instance.

## Architecture

```text
Client
  |
  v
Ingress
  |
  v
Kubernetes Service
  |
  v
NGINX Deployment
  |
  +--> ConfigMap
  +--> Secret
  +--> PersistentVolumeClaim

Metrics Server
  |
  v
Horizontal Pod Autoscaler
```

## Tech Stack

- Kubernetes
- Minikube
- kubectl
- Docker
- NGINX
- AWS EC2
- Linux
- Git

## Kubernetes Features Implemented

### Workload Management
- Namespace isolation using `dev`
- Deployment and ReplicaSet management
- Rolling updates and rollback support
- Self-healing through Kubernetes controllers

### Networking
- Kubernetes Service for stable Pod access
- Ingress for HTTP routing

### Configuration and Secrets
- ConfigMap for non-sensitive configuration
- Secret pattern using `secret.example.yaml`
- Real secret files excluded through `.gitignore`

### Reliability
- Liveness probes
- Readiness probes
- CPU and memory requests
- CPU and memory limits

### Storage
- PersistentVolumeClaim for persistent application data

### Scaling
The Horizontal Pod Autoscaler is configured with:

```text
Minimum replicas: 1
Maximum replicas: 5
Target CPU usage: 50%
```

Metrics are supplied through Kubernetes Metrics Server.

### Security
RBAC is implemented using:

```text
ServiceAccount → Role → RoleBinding
```

The `dev-reader` service account can inspect Pods but is not granted permission to delete them, demonstrating least-privilege access.

## Project Structure

```text
.
├── namespace.yaml
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── configmap.yaml
├── pvc.yaml
├── hpa.yaml
├── secret.example.yaml
├── serviceaccount.yaml
├── role.yaml
├── rolebinding.yaml
└── README.md
```

## Prerequisites

```bash
kubectl version --client
minikube version
docker --version
```

Start the cluster and required components:

```bash
minikube start --driver=docker
minikube addons enable ingress
minikube addons enable metrics-server
```

## Deployment

Create the namespace:

```bash
kubectl apply -f namespace.yaml
```

Create a local Secret from the example:

```bash
cp secret.example.yaml secret.yaml
# Edit placeholder values before applying
kubectl apply -f secret.yaml
```

Deploy the remaining resources:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f pvc.yaml
kubectl apply -f serviceaccount.yaml
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f hpa.yaml
```

## Verification

```bash
kubectl get pods -n dev
kubectl get deployments -n dev
kubectl get svc -n dev
kubectl get ingress -n dev
kubectl get pvc -n dev
kubectl get hpa -n dev
kubectl top pods -n dev
```

## RBAC Test

Confirm the service account can list Pods:

```bash
kubectl auth can-i list pods \
  --as=system:serviceaccount:dev:dev-reader \
  -n dev
```

Confirm it cannot delete Pods:

```bash
kubectl auth can-i delete pods \
  --as=system:serviceaccount:dev:dev-reader \
  -n dev
```

## Troubleshooting Commands

```bash
kubectl get pods -n dev -o wide
kubectl describe pod <pod-name> -n dev
kubectl logs <pod-name> -n dev
kubectl get events -n dev
kubectl rollout status deployment/nginx-deployment -n dev
kubectl rollout history deployment/nginx-deployment -n dev
```

## Skills Demonstrated

- Kubernetes Deployments and Services
- Ingress networking
- ConfigMaps and Secrets
- PersistentVolumeClaims
- Liveness and readiness probes
- Resource requests and limits
- Horizontal Pod Autoscaling
- Metrics Server
- Kubernetes RBAC
- Linux and Kubernetes troubleshooting

## Outcome

This repository demonstrates a complete Kubernetes application environment rather than only a simple container deployment. It brings together networking, configuration management, storage, scaling, reliability, security, and operational troubleshooting.

---

**Author:** Manoj Kumar  
**Focus:** Cloud & DevOps Engineering
