# ArgoCD Setup Guide

This guide will walk you through the process of setting up ArgoCD in your Kubernetes cluster and accessing the UI.

## Prerequisites

- Kubernetes cluster (v1.19+)
- kubectl command-line tool
- Network access to your Kubernetes API server

## Installation Steps

### 1. Create a namespace for ArgoCD

```bash
kubectl create namespace argocd
```

### 2. Apply the ArgoCD Installation Manifest

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This command installs ArgoCD with the non-HA (High Availability) version which is suitable for learning purposes.

### 3. Wait for ArgoCD Components to Start

```bash
kubectl get pods -n argocd
```

Wait until all pods are in a Running state. This may take a few minutes.

### 4. Access the ArgoCD UI

#### Option 1: Port-forwarding (Easiest for local development)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open your browser and navigate to: https://localhost:8080

#### Option 2: Expose with a Load Balancer (For cloud environments)

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Get the IP address:
```bash
kubectl get svc argocd-server -n argocd
```

### 5. Login to ArgoCD

For your initial login, use:
- Username: admin
- Password: (automatically generated)

To get the auto-generated password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

It's recommended to change the password after your first login.

### 6. Installing the ArgoCD CLI (Optional but useful)

#### For macOS:
```bash
brew install argocd
```

#### For Linux:
```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

#### For Windows:
Download the latest version from https://github.com/argoproj/argo-cd/releases/latest

### 7. CLI Login

```bash
argocd login localhost:8080
```

## What's Next?

After completing the installation, proceed to the [First Application](./first-application.md) guide to learn how to deploy your first application with ArgoCD.

## Troubleshooting

### Certificate Issues
When using port-forwarding, you might encounter certificate warnings in your browser. This is expected for the self-signed certificate. You can safely proceed.

### Connection Issues
If you can't connect to the ArgoCD server, ensure that:
- Your port-forwarding command is running
- Your kubectl is configured correctly
- All the ArgoCD pods are running

### Unauthorized Access
If you get unauthorized errors while trying to access the UI, double-check the password you retrieved from the secret.
