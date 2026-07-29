# FILE_NAME: troubleshoot-sync.md
# DESCRIPTION: Runbook — diagnose hub vs spoke sync failures
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbook: Troubleshoot sync failures

## Intent

Locate whether a failure is hub bootstrap, spoke root, addon generator, or NaaS path mismatch.

## Decision tree

```text
Is the resource a hub bootstrap / spoke Argo install?
  YES → debug on HUB (section A)
  NO  → is it spoke-addons / spoke-naas / naas-* / addon-* on spoke?
          YES → debug on SPOKE (section B)
          NO  → check leftover hub-push Applications (section C)
```

## A. Hub bootstrap / spoke install

```bash
kubectl -n argocd get app bootstrap-hub-addons
kubectl -n argocd get applicationset addons-argocd addons-spoke-root
kubectl -n argocd get app | grep -E 'argo-cd|spoke-root'
```

| Symptom | Checks |
| --- | --- |
| `bootstrap-hub-addons` Unknown/Missing | Re-apply `exclude/bootstrap.yaml`; repo URL reachable |
| No `spoke-root-<name>` | Hub cluster secret missing `enable_argocd=true` |
| Argo CD install OutOfSync | Helm values under `environments/.../argo-cd/`; CRD SSA options |
| Spoke API errors | Hub still needs kubeconfig for bootstrap destination |

## B. Spoke day-2

```bash
kubectl -n argocd get app spoke-addons spoke-naas
kubectl -n argocd get applicationset
kubectl -n argocd get secret in-cluster -o yaml | yq '.metadata|{labels,annotations}'
```

| Symptom | Checks |
| --- | --- |
| `spoke-addons` not Synced | Git credentials; path `bootstrap/control-plane/addons/spoke` |
| AppSet exists, no Applications | Missing `enable_*` **label** on `in-cluster` |
| Application uses wrong values | `addons_cluster_name` annotation missing/wrong (not `in-cluster`) |
| NaaS AppSet empty | `enable_naas` label; `naas_cluster_name` matches `env/*/<name>/` |
| Destination still `name: <cluster>` | Stale manifest — ensure spoke AppSets / NaaS v2 from current `main` |

### Confirm in-cluster destination

```bash
kubectl -n argocd get app naas-EXAMPLE -o jsonpath='{.spec.destination}{"\n"}'
# expect server: https://kubernetes.default.svc
```

## C. Leftover hub-push (migration)

On hub:

```bash
kubectl -n argocd get applicationset
kubectl -n argocd get app | grep -iE 'metrics-server|envoy|naas-'
```

If hub still has Applications deploying addons/NaaS with `destination.name` to spokes:

1. Confirm spoke already reconciles the same workload.
2. Delete hub Application / AppSet carefully (`preserveResourcesOnDeletion` may leave children).
3. Do not delete spoke-local copies first.

## D. Agent connectivity

If using Argo CD Agent principal on hub:

- Spoke agent pods Running; mTLS to principal
- Autonomous mode: spoke app-controller still reconciles when hub UI is down
- Hub “missing apps” is expected for day-2 — look at spoke Argo UI/CLI

## Collect for escalation

```bash
# Hub
kubectl -n argocd get app,applicationset -o wide
# Spoke
kubectl -n argocd get app,applicationset -o wide
kubectl -n argocd get secret in-cluster -o yaml | sed '/password\|token\|config:/d'
```

Never paste kubeconfig `config` / tokens into tickets.

## See also

- [../operations/day-2.md](../operations/day-2.md)
- [ADR-0001](../adr/0001-hub-spoke-argocd-agents.md)
