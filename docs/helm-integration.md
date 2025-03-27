# Using Helm Charts with ArgoCD

This guide explains how to integrate Helm charts with ArgoCD for managing complex applications.

## Introduction to Helm in ArgoCD

Helm is a package manager for Kubernetes that helps you define, install, and upgrade applications. ArgoCD has native support for Helm charts, allowing you to:

- Deploy applications packaged as Helm charts
- Override values during deployment
- Track changes to chart values and versions
- Manage chart releases through GitOps workflows

## Methods for Using Helm with ArgoCD

There are three main ways to use Helm charts with ArgoCD:

1. **Helm Charts from Repositories**: Use charts from public or private Helm repositories
2. **Git-based Helm Charts**: Store Helm charts in your Git repository
3. **Generated Manifests**: Use Helm to generate manifests and commit them to Git

## Method 1: Helm Charts from Repositories

### Creating an Application from a Helm Repository

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prometheus
  namespace: argocd
spec:
  project: default
  source:
    chart: prometheus
    repoURL: https://prometheus-community.github.io/helm-charts
    targetRevision: 15.10.1
    helm:
      values: |
        server:
          persistentVolume:
            enabled: false
        alertmanager:
          enabled: false
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Adding a Private Helm Repository

To use private Helm repositories, you need to add them to ArgoCD:

```bash
argocd repo add https://my-private-helm-repo.example.com \
  --name my-private-repo \
  --username myuser \
  --password mypassword
```

## Method 2: Git-based Helm Charts

You can store Helm charts directly in your Git repository:

```
repo/
├── charts/
│   └── my-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
```

Then reference it in your Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: charts/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

## Method 3: Generated Manifests

You can pre-render Helm templates and store the resulting manifests in Git:

```bash
# Generate manifests
helm template my-app ./charts/my-app -f values.yaml > manifests/my-app.yaml

# Commit to Git
git add manifests/my-app.yaml
git commit -m "Update my-app manifests"
git push
```

Then reference the generated manifest in your Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

## Overriding Helm Values

ArgoCD offers several ways to override Helm values:

### Inline Values

```yaml
spec:
  source:
    helm:
      values: |
        service:
          type: NodePort
        resources:
          limits:
            memory: 512Mi
```

### Values Files from the Repository

```yaml
spec:
  source:
    helm:
      valueFiles:
        - values.yaml
        - values-production.yaml
```

### Parameter Overrides

```yaml
spec:
  source:
    helm:
      parameters:
        - name: service.type
          value: NodePort
        - name: image.tag
          value: latest
```

## Value Files with Environment-specific Overrides

You can maintain different value files for different environments:

```
environments/
├── dev/
│   └── values.yaml
├── staging/
│   └── values.yaml
└── production/
    └── values.yaml
```

```yaml
spec:
  source:
    path: charts/my-app
    helm:
      valueFiles:
        - values.yaml
        - ../../environments/production/values.yaml
```

## Advanced Helm Features in ArgoCD

### Helm Hooks

ArgoCD supports Helm hooks for specialized operations during deployment:

```yaml
metadata:
  annotations:
    helm.sh/hook: pre-install
    helm.sh/hook-weight: "5"
```

### Release Name Override

By default, ArgoCD uses the application name as the Helm release name, but you can override it:

```yaml
spec:
  source:
    helm:
      releaseName: custom-release-name
```

### Value Files from ConfigMaps

You can reference ConfigMaps for your Helm values:

```yaml
spec:
  source:
    helm:
      valueFiles:
        - $values/config-map-name/key-name
```

## Helm Values Management Best Practices

1. **Keep Base Values Simple**: Define sensible defaults in your chart's values.yaml
2. **Environment-specific Overrides**: Use separate value files for different environments
3. **Use Git for Values**: Store all value overrides in Git for traceability
4. **Minimize Inline Values**: Prefer value files over inline values for better readability
5. **Version Your Charts**: Use semantic versioning for your Helm charts
6. **Document Value Options**: Clearly document the available configuration options

## Example: Complete Helm Application

Here's a complete example of a Helm-based application in ArgoCD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/marvintv/argocd-learning-project.git
    targetRevision: HEAD
    path: charts/my-app
    helm:
      releaseName: my-custom-release
      valueFiles:
        - values.yaml
        - ../../environments/production/values.yaml
      parameters:
        - name: image.tag
          value: v1.2.3
      skipCrds: false
      ignoreMissingValueFiles: true
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Troubleshooting Helm Charts in ArgoCD

### Check Helm Release Status

```bash
argocd app get my-app
kubectl get secret -n my-app -l owner=helm
```

### View Generated Manifests

```bash
argocd app manifests my-app
```

### Debug Helm Template Rendering

```bash
argocd app get my-app --refresh-hard
argocd app wait my-app --operation
```

## Next Steps

After mastering Helm integration with ArgoCD, you might want to explore:
- [Kustomize Integration](./kustomize-integration.md)
- [Multi-cluster Deployments](./multi-cluster.md)
- [Application Health Monitoring](./application-health.md)

Integrating Helm with ArgoCD combines the flexibility of Helm's templating with the GitOps workflow of ArgoCD, giving you a powerful way to manage complex applications.
