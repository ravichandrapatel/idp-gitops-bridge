# FILE_NAME: provision-naas-tenant.md
# DESCRIPTION: Runbook — provision and verify a NaaS tenant on a spoke
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbook: Provision a NaaS tenant

## Intent

Create a Namespace-as-a-Service tenant so the **spoke** ApplicationSet installs the OCI chart in-cluster.

## Preconditions

- Spoke has `enable_naas=true` **label** and `naas_cluster_name=<logical>` **annotation** on `in-cluster`
- Application `spoke-naas` Synced; ApplicationSet `namespace-as-service` present
- Backstage template from [idp-backstage-templates](https://github.com/ravichandrapatel/idp-backstage-templates) (tag `template-namespace-as-service-v1.1.1` or newer)
- Catalog Resource name equals logical cluster name

## Happy path (Backstage)

1. Open **Namespace as a Service** in the portal.
2. Select cluster EntityPicker value → name must match `naas_cluster_name`.
3. Submit → PR to `idp-argocd-apps` at `env/<env>/<cluster>/<tenant>.yaml`.
4. Auto-merge workflow merges `feat(naas):` / `backstage/naas-*` branches.
5. On spoke, Application `naas-<tenant>` appears and syncs to namespace `<tenant>-devops`.

## Manual Git path (break-glass)

```yaml
# env/nonprod/labs/payments.yaml  (example)
tenant: payments
env: nonprod
clusterName: labs
chartVersion: "1.0.0"
project: default
# ... helmValues block per skeleton
```

Merge to `main` on idp-argocd-apps. Spoke git generator path:

```text
*/{{metadata.annotations.naas_cluster_name}}/*.yaml
```

## Verification

```bash
# Spoke
kubectl -n argocd get app naas-payments
kubectl get ns | grep '^payments'
```

| Check | Pass |
| --- | --- |
| App exists only on **spoke** | Not created on hub with remote destination |
| Destination | `https://kubernetes.default.svc` |
| Chart | `ghcr.io/ravichandrapatel/charts` / `namespace-as-service` / `chartVersion` |

## Common failures

| Symptom | Likely cause |
| --- | --- |
| No Application generated | Missing `enable_naas` label or wrong `naas_cluster_name` |
| Generated on wrong cluster | Tenant file `clusterName` / path segment mismatch |
| Chart pull fail | OCI auth / wrong `chartVersion` |
| Hub shows old remote apps | Leftover hub-pushed Applications — delete after migration |

## Rollback / delete tenant

1. Delete `env/<env>/<cluster>/<tenant>.yaml` (PR) — AppSet prune should remove Application if prune enabled.
2. Confirm namespaces and finalizers; clean stuck finalizers only with care.

## See also

- [idp-argocd-apps docs](https://github.com/ravichandrapatel/idp-argocd-apps)
- [troubleshoot-sync.md](troubleshoot-sync.md)
