# Deploying Your First Application with ArgoCD

This guide will walk you through deploying your first application using ArgoCD. We'll deploy a simple Nginx web server to demonstrate the basic workflow.

## Prerequisites

- ArgoCD installed in your Kubernetes cluster (see the [Setup Guide](./setup.md))
- Access to the ArgoCD UI or CLI

## Step 1: Create a Git Repository for Your Application

ArgoCD uses Git repositories as the source of truth for application definitions. For this example, we'll use a simple Nginx deployment.

Create a directory structure in this repository:

```
applications/
  nginx-demo/
    deployment.yaml
    service.yaml
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

## Step 2: Create an Application in ArgoCD

### Using the UI

1. Open the ArgoCD UI in your browser
2. Click on "New App" button
3. Fill in the application details:
   - Application Name: `nginx-demo`
   - Project: `default`
   - Sync Policy: `Manual`
   - Repository URL: Your repository URL (e.g., `https://github.com/marvintv/argocd-learning-project`)
   - Revision: `HEAD` (this will use the latest commit)
   - Path: `applications/nginx-demo`
   - Destination: Your Kubernetes cluster (usually `https://kubernetes.default.svc` for in-cluster)
   - Namespace: `default` (or create a new one)
4. Click "Create" to save the application

### Using the CLI

```bash
argocd app create nginx-demo \
  --repo https://github.com/marvintv/argocd-learning-project \
  --path applications/nginx-demo \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

## Step 3: Sync Your Application

After creating the application, ArgoCD will show it as "OutOfSync" because the resources don't exist in the cluster yet.

### Using the UI

1. In the ArgoCD UI, click on your application
2. Click the "Sync" button
3. Confirm the resources to sync
4. Click "Synchronize"

### Using the CLI

```bash
argocd app sync nginx-demo
```

## Step 4: Verify the Application

You should now see your application as "Synced" in the ArgoCD UI. To verify the deployment:

```bash
kubectl get pods,svc -l app=nginx-demo
```

## Step 5: Make a Change to Your Application

Let's modify the application to see how GitOps works:

1. Change the `deployment.yaml` file in your repository to increase the replicas to 3
2. Commit and push your changes
3. In the ArgoCD UI, you'll see that the application is now "OutOfSync"
4. Click "Sync" to apply the changes

## Understanding the ArgoCD Workflow

This simple example demonstrates the core GitOps workflow with ArgoCD:

1. **Define** - You define your desired state in Git
2. **Commit** - You commit and push changes to Git
3. **Sync** - ArgoCD detects differences and syncs the changes

## Auto-Sync (Optional)

You can configure ArgoCD to automatically sync changes. To enable this:

### Using the UI

1. Go to your application
2. Click the "App Details" button
3. Click "Edit"
4. Change "Sync Policy" to "Automatic"
5. Optionally enable "Self Heal" and "Prune"
6. Click "Save"

### Using the CLI

```bash
argocd app set nginx-demo --sync-policy automated
```

With auto-sync enabled, ArgoCD will automatically apply changes whenever you update your Git repository.

## Next Steps

Now that you've deployed your first application, you can explore more advanced features of ArgoCD:

- [Sync Policies and Strategies](./sync-policies.md)
- [Using Helm Charts](./helm-integration.md)
- [App of Apps Pattern](./app-of-apps.md)

Congratulations on deploying your first application with ArgoCD!
