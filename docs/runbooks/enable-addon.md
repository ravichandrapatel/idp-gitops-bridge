# FILE_NAME: enable-addon.md
# DESCRIPTION: Runbook — enable or disable a spoke platform addon
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbook: Enable or disable a spoke addon

## Intent

Turn a platform addon on or off for one spoke without hub-pushing the chart.

## Preconditions

- Spoke onboarded ([onboard-spoke.md](onboard-spoke.md))
- Spoke root Application `spoke-addons` Healthy
- Matching ApplicationSet exists under `bootstrap/control-plane/addons/spoke/`
- Helm values prepared under:
  - `environments/default/addons/<chart>/values.yaml`
  - optional `environments/<env>/addons/<chart>/values.yaml`
  - `environments/clusters/<logical>/addons/<chart>/values.yaml`

## Enable

### 1. Confirm chart / label mapping

| ApplicationSet | Label | Typical chart |
| --- | --- | --- |
| `addons-metrics-server` | `enable_metrics_server=true` | metrics-server |
| `addons-envoy-gateway` | `enable_envoy_gateway=true` | gateway-helm |
| `addons-gitea` | `enable_gitea=true` | gitea |
| `addons-vault` | `enable_vault=true` | vault |
| `addons-external-secrets` | `enable_external_secrets=true` | external-secrets |
| `addons-cloudnative-pg` | `enable_cnpg=true` | cloudnative-pg |
| `addons-backstage` | `enable_backstage=true` | backstage |
| `addons-keycloak` | `enable_keycloak=true` | keycloak |
| `addons-tekton` | `enable_tekton=true` | tekton |

### 2. Set label on spoke `in-cluster` secret

```bash
kubectl -n argocd label secret in-cluster enable_metrics_server=true --overwrite
```

### 3. Confirm annotation for values path

```bash
kubectl -n argocd get secret in-cluster -o jsonpath='{.metadata.annotations.addons_cluster_name}{"\n"}'
# expect logical name, e.g. labs
```

### 4. Commit values if needed

Edit `environments/clusters/<logical>/addons/<chart>/values.yaml` in this repo; merge to the revision the spoke tracks (`addons_repo_revision`, usually `main`).

### 5. Verify Application

```bash
kubectl -n argocd get applications | grep -i metrics-server
argocd app get <app-name> --grpc-web   # if CLI configured on spoke
```

## Disable

```bash
kubectl -n argocd label secret in-cluster enable_metrics_server-  # remove label
```

ApplicationSet stops generating the Application. Whether resources are pruned depends on the AppSet/Application `syncPolicy` (`prune` settings). Prefer explicit teardown for stateful addons (Vault, CNPG).

## Hub must not

- Add the enable label only on the hub spoke secret expecting day-2 push — hub no longer syncs `addons/spoke/`
- Point hub Applications at spoke addon charts with `destination.name` for day-2

## See also

- [troubleshoot-sync.md](troubleshoot-sync.md)
- [../operations/day-2.md](../operations/day-2.md)
