# FILE_NAME: bootstrap-hub.md
# DESCRIPTION: Runbook — apply and verify hub-only bootstrap
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbook: Bootstrap hub Argo CD control plane

## Intent

Install or refresh the hub Application that syncs `bootstrap/control-plane/addons/hub/` (Argo CD + spoke-root AppSets) into the hub cluster only.

## Preconditions

- `kubectl` context points at the **hub** cluster
- Argo CD installed in namespace `argocd`
- Network access from hub to `https://github.com/ravichandrapatel/idp-gitops-bridge.git`

## Steps

### 1. Apply bootstrap ApplicationSet

```bash
kubectl apply -f bootstrap/control-plane/exclude/bootstrap.yaml
```

This creates ApplicationSet `bootstrap-addons` → Application `bootstrap-hub-addons` with:

- destination: `https://kubernetes.default.svc` / `argocd`
- source path: `bootstrap/control-plane/addons/hub`

### 2. Wait for hub AppSets

```bash
kubectl -n argocd get applicationset bootstrap-addons
kubectl -n argocd get application bootstrap-hub-addons
kubectl -n argocd get applicationset addons-argocd addons-spoke-root
```

Expected: `addons-argocd` and `addons-spoke-root` exist on the hub.

### 3. Confirm hub is not loading spoke AppSets from oss/

```bash
kubectl -n argocd get applicationset | grep -E 'metrics-server|envoy|oss' || true
```

Spoke addon ApplicationSets should **not** be created by hub bootstrap. They appear only on spokes after root Apps sync.

## Verification

| Check | Pass criteria |
| --- | --- |
| `bootstrap-hub-addons` | Synced / Healthy |
| Hub AppSets | Only hub-scoped sets from `addons/hub/` |
| Hub workloads | Still managed separately under `workloads/` if used |

## Rollback

```bash
kubectl -n argocd delete application bootstrap-hub-addons --wait=false
kubectl -n argocd delete applicationset bootstrap-addons
# Re-apply previous commit of exclude/bootstrap.yaml if needed
```

`preserveResourcesOnDeletion` may leave child AppSets; delete explicitly if required.

## See also

- [onboard-spoke.md](onboard-spoke.md)
- [../architecture/hub-spoke.md](../architecture/hub-spoke.md)
