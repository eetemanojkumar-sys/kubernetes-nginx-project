# Kubernetes Nginx Deployment Project

## Overview

This project demonstrates the deployment of an Nginx web application on Kubernetes using production-oriented Kubernetes concepts such as Deployments, Services, Ingress, persistent storage, health probes, resource management, autoscaling, configuration management, Secrets, and RBAC.

The project was built and tested using Minikube running on an AWS EC2 instance.

## Architecture

```text
                        Client
                          |
                          v
                       Ingress
                          |
                          v
                    Nginx Service
                          |
                          v
                +-------------------+
                |    Deployment     |
                |                   |
                |    Nginx Pods     |
                +-------------------+
                          |
              +-----------+-----------+
              |                       |
              v                       v
          ConfigMap                 Secret
                                     
                          |
                          v
                         PVC
                          |
                          v
                  Persistent Storage
```

Metrics Server collects resource metrics and the Horizontal Pod Autoscaler uses CPU utilization to automatically adjust the number of Nginx replicas.

## Technologies Used

* Kubernetes
* Minikube
* kubectl
* Docker
* Nginx
* AWS EC2
* Git
* Linux

## Kubernetes Concepts Implemented

### Namespace

Application resources are deployed inside the `dev` namespace to provide logical isolation.

### Deployment

The Nginx application is managed using a Kubernetes Deployment.

The Deployment provides:

* Declarative application management
* Replica management
* Rolling updates
* Self-healing
* Rollback support

### Service

A Kubernetes Service exposes the Nginx Pods through a stable network endpoint.

The Service uses labels and selectors to discover the appropriate application Pods.

### Ingress

Ingress provides HTTP routing to the Nginx Service.

Traffic flow:

```text
Client
  |
Ingress
  |
Service
  |
Pods
```

### ConfigMap

Non-sensitive application configuration is stored using a Kubernetes ConfigMap and injected into the application container.

### Secret

Sensitive configuration is managed using a Kubernetes Secret.

The real `secret.yaml` file is excluded from Git using `.gitignore`.

`secret.example.yaml` demonstrates the expected Secret structure using placeholder values.

### Persistent Storage

Persistent application storage is implemented using a PersistentVolumeClaim.

The PVC is mounted into:

```text
/usr/share/nginx/html
```

This allows application data to persist when a Pod is deleted and recreated.

### Liveness Probe

The liveness probe checks whether the Nginx container remains healthy.

If the liveness probe repeatedly fails, Kubernetes can restart the container.

### Readiness Probe

The readiness probe determines whether the application is ready to receive traffic.

Pods that fail readiness checks are removed from ready Service endpoints until they recover.

### Resource Requests and Limits

CPU and memory requests and limits are configured for the Nginx container.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "64Mi"
  limits:
    cpu: "500m"
    memory: "128Mi"
```

Requests help Kubernetes schedule Pods.

Limits control the maximum resources available to the container.

### Horizontal Pod Autoscaler

The project uses HPA to automatically scale the Nginx Deployment according to CPU utilization.

Configuration:

```text
Minimum replicas: 1
Maximum replicas: 5
Target CPU:       50%
```

Metrics are supplied through Kubernetes Metrics Server.

### RBAC

Role-Based Access Control is implemented using:

```text
ServiceAccount
      |
      v
RoleBinding
      |
      v
Role
```

The `dev-reader` ServiceAccount is allowed to:

* Get Pods
* List Pods
* Watch Pods

It is not allowed to delete Pods.

This demonstrates the principle of least privilege.

## Project Structure

```text
kubernetes-nginx-project/
|
├── README.md
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
└── .gitignore
```

## Prerequisites

Install:

* kubectl
* Minikube
* Docker
* Git

Start Minikube:

```bash
minikube start --driver=docker
```

Enable Ingress:

```bash
minikube addons enable ingress
```

Enable Metrics Server:

```bash
minikube addons enable metrics-server
```

## Deployment

Clone the repository and enter the project directory.

Create the namespace first:

```bash
kubectl apply -f namespace.yaml
```

Create your local Secret from the example:

```bash
cp secret.example.yaml secret.yaml
```

Edit the placeholder values:

```bash
nano secret.yaml
```

Apply the Secret:

```bash
kubectl apply -f secret.yaml
```

Deploy the remaining Kubernetes resources.

For example:

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

Check Pods:

```bash
kubectl get pods -n dev
```

Check Deployment:

```bash
kubectl get deployments -n dev
```

Check Service:

```bash
kubectl get svc -n dev
```

Check Ingress:

```bash
kubectl get ingress -n dev
```

Check persistent storage:

```bash
kubectl get pvc -n dev
kubectl get pv
```

Check HPA:

```bash
kubectl get hpa -n dev
```

Check resource usage:

```bash
kubectl top pods -n dev
kubectl top nodes
```

## Testing RBAC

Verify that `dev-reader` can list Pods:

```bash
kubectl auth can-i list pods \
  --as=system:serviceaccount:dev:dev-reader \
  -n dev
```

Expected:

```text
yes
```

Verify that it cannot delete Pods:

```bash
kubectl auth can-i delete pods \
  --as=system:serviceaccount:dev:dev-reader \
  -n dev
```

Expected:

```text
no
```

## Troubleshooting Commands

Useful Kubernetes troubleshooting commands used during this project:

```bash
kubectl get pods -n dev
kubectl get pods -n dev -o wide
kubectl describe pod <pod-name> -n dev
kubectl logs <pod-name> -n dev
kubectl get events -n dev
kubectl describe deployment nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev
kubectl rollout history deployment/nginx-deployment -n dev
```

## Key Learning Outcomes

Through this project I practiced:

* Kubernetes cluster fundamentals
* Pods and Deployments
* ReplicaSets
* Namespaces
* Labels and selectors
* ClusterIP and NodePort Services
* Ingress routing
* ConfigMaps and Secrets
* PersistentVolumeClaims
* Persistent storage
* Liveness and readiness probes
* CPU and memory requests/limits
* Metrics Server
* Horizontal Pod Autoscaling
* ServiceAccounts
* Kubernetes RBAC
* Kubernetes YAML manifests
* Kubernetes troubleshooting

## Security

Sensitive credentials should never be committed to source control.

The project uses:

```text
secret.example.yaml → safe placeholder configuration
secret.yaml         → local Secret ignored by Git
```

Always use appropriate secret-management solutions for real production environments.

## Future Improvements

Potential improvements include:

* Deploying the application to Amazon EKS
* Using AWS EBS CSI for persistent storage
* Installing Prometheus and Grafana for monitoring
* Adding Helm charts
* Implementing CI/CD using Jenkins
* Using HTTPS/TLS with Ingress
* Integrating centralized logging
* Implementing GitOps using Argo CD

## Author

Eete Manoj Kumar

DevOps / Cloud Engineering Project
