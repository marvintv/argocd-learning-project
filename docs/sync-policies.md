# ArgoCD Sync Policies and Strategies

Sync policies and strategies are essential aspects of ArgoCD that control how your applications are kept in sync with their Git-defined desired state. This guide explains the various options and their use cases.

## Sync Policies

Sync policies define when and how ArgoCD synchronizes your applications.

### Manual Sync

By default, ArgoCD uses manual synchronization, meaning you need to trigger a sync operation manually through the UI or CLI.

**When to use**:
- During initial testing and learning
- When you want explicit control over when changes are applied
- For production environments where changes should be reviewed before application

### Automated Sync

With automated sync, ArgoCD will automatically apply changes when it detects differences between the Git repository and the cluster state.

```bash
argocd app set <app-name> --sync-policy automated
```

**When to use**:
- Development environments where rapid iteration is valuable
- CI/CD pipelines where changes are already reviewed before commit
- When you want true GitOps automation

### Prune Resources

By default, even with automated sync, ArgoCD won't delete resources that exist in the cluster but not in Git. Enabling prune allows these orphaned resources to be removed.

```bash
argocd app set <app-name> --sync-policy automated --auto-prune
```

**When to use**:
- When you want complete consistency with your Git definitions
- When you're confident in your Git being the sole source of truth
- In environments where accidental resource retention could cause issues

### Self Healing

Self-healing monitors the live state and automatically syncs when it detects that the live state has drifted from the desired state, even if the Git repository hasn't changed.

```bash
argocd app set <app-name> --sync-policy automated --self-heal
```

**When to use**:
- To prevent manual changes to the cluster
- In environments where compliance and consistency are critical
- When you want to ensure that manual debugging doesn't persist

## Sync Options

Sync options provide fine-grained control over how sync operations behave.

### Selective Sync

You can exclude specific resources from sync operations:

```bash
argocd app set <app-name> --ignore-resource-types ConfigMap,Secret
```

**When to use**:
- When certain resources need manual management
- For sensitive resources like Secrets that might be managed elsewhere
- When specific resources need specialized handling

### Apply Only

By default, ArgoCD uses `kubectl apply` to apply changes. You can configure it to only apply changes without pruning:

```bash
argocd app set <app-name> --sync-option ApplyOnly=true
```

**When to use**:
- When you want to ensure no resources are deleted during sync
- During initial setup phases
- When transitioning existing applications to ArgoCD

### Replace

Instead of using `kubectl apply`, you can configure ArgoCD to use `kubectl replace`:

```bash
argocd app set <app-name> --sync-option Replace=true
```

**When to use**:
- When you need to completely replace resources rather than patch them
- For resources that don't merge well with apply
- When you want to ensure no existing fields are preserved

### Retry

You can configure the number of retry attempts for sync operations:

```bash
argocd app set <app-name> --sync-option Retry=5
```

**When to use**:
- For resources that might have timing dependencies
- When deploying to environments with potential transient issues
- For complex applications where ordering might cause temporary failures

## Sync Waves and Hooks

ArgoCD supports more advanced synchronization patterns using waves and hooks.

### Sync Waves

Sync waves allow you to control the order of resource creation/deletion through annotations:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "5"
```

Resources are synced in ascending wave order (negative waves first, then 0, then positive).

**When to use**:
- To ensure proper dependency ordering (e.g., databases before applications)
- For complex applications with specific startup requirements
- When certain resources must be created before others

### Resource Hooks

Hooks let you run jobs or operations at specific points in the sync process:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
```

Available hooks include:
- `PreSync`: Run before syncing
- `Sync`: Run during syncing
- `PostSync`: Run after syncing
- `SyncFail`: Run if the sync operation fails

**When to use**:
- For database migrations (PreSync)
- For smoke tests (PostSync)
- For cleanup operations (SyncFail)
- For specialized initialization tasks

## Example: Complete Sync Policy Setup

Here's an example of setting up a comprehensive sync policy:

```bash
argocd app set myapp \
  --sync-policy automated \
  --auto-prune \
  --self-heal \
  --sync-option CreateNamespace=true \
  --sync-option PrunePropagationPolicy=foreground \
  --sync-option PruneLast=true
```

This configuration:
- Automatically syncs when changes are detected
- Removes resources that are no longer in Git
- Reverts manual changes to the cluster
- Creates the target namespace if it doesn't exist
- Uses foreground propagation for pruning (waits for dependents to be deleted)
- Prunes resources last (after successful creation of new resources)

## Next Steps

After understanding sync policies and strategies, you might want to explore:
- [Using Helm Charts with ArgoCD](./helm-integration.md)
- [The App of Apps Pattern](./app-of-apps.md)

By mastering sync policies, you can achieve the right balance of automation and control for your GitOps workflows.
