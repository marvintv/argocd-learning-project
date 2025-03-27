# The App of Apps Pattern in ArgoCD

The "App of Apps" pattern is a powerful technique in ArgoCD that allows you to manage multiple applications as a single unit. This guide explains the concept, benefits, and implementation of this pattern.

## What is the App of Apps Pattern?

In the App of Apps pattern, you create a parent ArgoCD Application that deploys and manages other ArgoCD Applications. These child applications can then deploy actual resources to your cluster.

Think of it like:
- **Parent Application**: Manages the definitions of child applications
- **Child Applications**: Manage actual Kubernetes resources

## Why Use App of Apps?

This pattern offers several advantages:

1. **Centralized Management**: Manage your entire application ecosystem from a single Git repository
2. **Environment Consistency**: Easily replicate the same set of applications across multiple environments
3. **Simplified Onboarding**: New team members can understand the full system by looking at a single repository
4. **Dependency Management**: Control the order of application deployment
5. **Scalability**: Easily add or remove applications from your ecosystem

## How to Implement the App of Apps Pattern

### 1. Create a Directory Structure

```
root/
├── apps/                 # Parent app directory
│   ├── Chart.yaml        # If using Helm
│   ├── values.yaml       # If using Helm
│   └── templates/        # Application definitions go here
│       ├── app1.yaml
│       ├── app2.yaml
│       └── app3.yaml
├── app1/                 # Child app directories
├── app2/
└── app3/
```

### 2. Create Application Definitions

For each child application, create an Application manifest like this:

```yaml
# apps/templates/app1.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: app1
  destination:
    server: https://kubernetes.default.svc
    namespace: app1
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 3. Create the Parent Application

Create the parent application that will manage all child applications:

```yaml
# parent-app.yaml (applied manually or through CI/CD)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: all-apps
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 4. Apply the Parent Application

```bash
kubectl apply -f parent-app.yaml
```

Once applied, ArgoCD will create all the child applications defined in the templates.

## Example Implementation

Let's create a simple example with three applications: nginx, redis, and prometheus.

### Project Structure

```
argocd-learning-project/
├── apps/
│   └── templates/
│       ├── nginx.yaml
│       ├── redis.yaml
│       └── prometheus.yaml
├── nginx-app/
│   ├── deployment.yaml
│   └── service.yaml
├── redis-app/
│   ├── deployment.yaml
│   └── service.yaml
└── prometheus-app/
    ├── deployment.yaml
    └── service.yaml
```

### Child Application Definitions

```yaml
# apps/templates/nginx.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: nginx-app
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
---
# apps/templates/redis.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: redis
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: redis-app
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Parent Application Definition

```yaml
# parent-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: all-apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Advanced App of Apps Techniques

### Using Helm for App of Apps

You can use Helm to template your applications, allowing you to parameterize their configuration:

```yaml
# apps/templates/application.yaml
{{- range .Values.applications }}
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: {{ .name }}
  namespace: argocd
spec:
  project: default
  source:
    repoURL: {{ $.Values.repoURL }}
    targetRevision: {{ $.Values.targetRevision }}
    path: {{ .path }}
  destination:
    server: https://kubernetes.default.svc
    namespace: {{ .namespace | default "default" }}
  syncPolicy:
    automated:
      prune: {{ .prune | default true }}
      selfHeal: {{ .selfHeal | default true }}
{{- end }}
```

With a values file like:

```yaml
# apps/values.yaml
repoURL: https://github.com/marvintv/argocd-learning-project.git
targetRevision: HEAD

applications:
  - name: nginx
    path: nginx-app
    namespace: demo
  - name: redis
    path: redis-app
    namespace: demo
  - name: prometheus
    path: prometheus-app
    namespace: monitoring
    prune: false
```

### Handling Dependencies with Sync Waves

You can control the order of application deployment using sync waves:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: database
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "-1"  # Deploy before other apps
spec:
  # ... rest of definition
```

### Multi-Environment App of Apps

For different environments, you can have separate directories or branches:

```
├── apps/
│   ├── dev/
│   │   └── templates/
│   ├── staging/
│   │   └── templates/
│   └── production/
│       └── templates/
```

## Considerations and Best Practices

1. **Repository Layout**: Design your repository structure to clearly separate parent and child applications
2. **Sync Policies**: Consider which applications should be auto-synced and which should be manually controlled
3. **Namespace Management**: Decide whether to use a single namespace or multiple namespaces for your applications
4. **RBAC**: Set up appropriate RBAC to control who can manage which applications
5. **Deletion Handling**: Use finalizers to ensure proper cleanup when applications are deleted

## Next Steps

After mastering the App of Apps pattern, you might want to explore:
- [Using Helm Charts with ArgoCD](./helm-integration.md)
- [GitOps Workflows](./gitops-workflows.md)
- [ArgoCD Scaling Strategies](./scaling-strategies.md)

The App of Apps pattern is one of the most powerful techniques for managing complex application deployments with ArgoCD and GitOps principles.
