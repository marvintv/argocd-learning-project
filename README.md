# ArgoCD Learning Project

This repository is designed as a hands-on learning project for ArgoCD and GitOps practices. 

## What is ArgoCD?

[ArgoCD](https://argoproj.github.io/argo-cd/) is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment of desired application states in the specified target environments. Application deployments can track updates to branches, tags, or pinned to a specific version of manifests at a Git commit.

## Why GitOps with ArgoCD?

GitOps is a way to do Kubernetes cluster management and application delivery. It works by using Git as a single source of truth for declarative infrastructure and applications. With GitOps:

- The entire system is described declaratively
- The canonical desired system state is versioned in Git
- Approved changes can be automatically applied to the system
- Software agents ensure correctness and alert on divergence

## Learning Paths

This repository is organized to help you learn ArgoCD through progressive examples:

1. **Basic Setup** - Installing ArgoCD in a Kubernetes cluster
2. **Simple Application** - Deploying your first application with ArgoCD
3. **Sync Policies** - Understanding how ArgoCD keeps your applications in sync
4. **Application of Applications (App of Apps)** - Managing multiple applications
5. **Helm and Kustomize** - Using ArgoCD with Helm charts and Kustomize
6. **Advanced Patterns** - Learning about progressive delivery, canary deployments, etc.

## Prerequisites

- A Kubernetes cluster (local like Minikube, Kind or remote)
- kubectl configured to communicate with your cluster
- Git installed on your local machine
- Basic understanding of Kubernetes concepts

## Getting Started

Check out the [Setup Guide](./docs/setup.md) to install ArgoCD and get started with your learning journey.

Happy learning!
