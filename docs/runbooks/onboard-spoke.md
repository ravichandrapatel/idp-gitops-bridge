# FILE_NAME: onboard-spoke.md
# DESCRIPTION: Runbook — onboard a spoke cluster for agent / Argo CD day-2
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbook: Onboard a spoke cluster

## Intent

Register a spoke so the hub installs Argo CD (bootstrap) and spoke root Applications; then configure the spoke `in-cluster` secret for addon + NaaS reconciliation.

## Preconditions

- Hub bootstrap already applied ([bootstrap-hub.md](bootstrap-hub.md))
- Spoke API reachable from hub for **bootstrap** Applications (`destination.name`)
- Logical cluster name chosen (e.g. `labs`) — used in Git paths and Backstage catalog
- Values tree exists or will exist at `environments/clusters/<logical>/addons/`

## Steps

### 1. Register spoke on hub Argo CD

Create/update the hub cluster Secret (or declarative cluster registration) so Argo knows the spoke as `<argo-cluster-name>` (often the logical name, e.g. `labs`).

Ensure labels/annotations used by hub generators:

```text
# Label on hub's spoke cluster secret
enable_argocd=true

# Annotations (repo pins for hub AppSets that render Applications)
addons_repo_url=https://github.com/ravichandrapatel/idp-gitops-bridge.git
addons_repo_revision=main
addons_repo_basepath=
```

### 2. Wait for hub to install spoke Argo CD

```bash
# On hub
kubectl -n argocd get app -l app.kubernetes.io/instance 2>/dev/null || true
kubectl -n argocd get applications | grep -E "addon-.*-argo-cd|spoke-root-"
```

Expect Applications similar to:

- `addon-<name>-argo-cd`
- `spoke-root-<name>`

### 3. Configure spoke local `in-cluster` secret

On the **spoke** Argo CD (after it is up), label/annotate the local cluster secret (usually `in-cluster`):

```bash
LOGICAL=labs   # must match Git path segments

kubectl -n argocd label secret in-cluster \
  enable_argocd=true \
  enable_naas=true \
  enable_metrics_server=true \
  --overwrite

kubectl -n argocd annotate secret in-cluster \
  addons_cluster_name="${LOGICAL}" \
  naas_cluster_name="${LOGICAL}" \
  addons_repo_url=https://github.com/ravichandrapatel/idp-gitops-bridge.git \
  addons_repo_revision=main \
  addons_repo_basepath= \
  --overwrite
```

Add further `enable_*=true` labels for each addon the spoke should run.

### 4. Verify spoke root Applications

On the spoke:

```bash
kubectl -n argocd get application spoke-addons spoke-naas
kubectl -n argocd get applicationset
```

Expect ApplicationSets from `addons/spoke/` and `namespace-as-service` after sync.

### 5. Catalog + Git paths

- Backstage: Resource `metadata.name` = logical name ([idp-backstage-templates](https://github.com/ravichandrapatel/idp-backstage-templates) `catalog/`)
- Bridge values: `environments/clusters/<logical>/addons/<chart>/values.yaml`
- NaaS tenants: `env/<env>/<logical>/<tenant>.yaml` in idp-argocd-apps

### 6. Agent principal (autonomous)

Follow [Argo CD Agent](https://argocd-agent.readthedocs.io/stable/concepts/architecture/) to register the spoke agent with the hub principal (mTLS). Hub observability of the agent does **not** replace spoke-local sync ownership.

## Verification checklist

- [ ] Hub Application `spoke-root-<name>` Healthy
- [ ] Spoke `spoke-addons` / `spoke-naas` Synced
- [ ] Addon App appears when corresponding `enable_*` label is set
- [ ] Test NaaS file under `env/<env>/<logical>/` creates `naas-<tenant>` on spoke only

## Rollback

1. Remove `enable_argocd=true` from hub spoke secret (stops new root/argo apps; existing may remain).
2. Delete hub Applications `spoke-root-<name>` and `addon-<name>-argo-cd` if decommissioning.
3. On spoke, delete Applications/ApplicationSets if wiping day-2 (destructive).

## See also

- [enable-addon.md](enable-addon.md)
- [provision-naas-tenant.md](provision-naas-tenant.md)
- [troubleshoot-sync.md](troubleshoot-sync.md)
