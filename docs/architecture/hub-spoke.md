# FILE_NAME: hub-spoke.md
# DESCRIPTION: Hub–spoke Argo CD Agent architecture overview
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Hub–spoke architecture

**ADR:** [ADR-0001](../adr/0001-hub-spoke-argocd-agents.md) · **Diagrams:** [diagrams.md](diagrams.md) · **Day-2:** [../operations/day-2.md](../operations/day-2.md)

## Goal

| Plane | Owns | Does not own |
| --- | --- | --- |
| **Hub Argo CD** + this repo | Cluster inventory; bootstrap spoke Argo CD / agent; spoke root Apps; hub in-cluster workloads | Spoke namespaces, NaaS tenants, spoke addons after bootstrap |
| **Spoke Agent (autonomous)** | Addons + NaaS + platform config **in that cluster** | Creating other clusters |
| **idp-argocd-apps** | `env/<env>/<cluster>/<tenant>.yaml` | Hub ApplicationSets with remote `destination.name` for NaaS |
| **idp-backstage-templates** | Form → PR into idp-argocd-apps | Talking to hub for tenant sync |

## Repository map

```text
idp-gitops-bridge/
  bootstrap/control-plane/
    exclude/bootstrap.yaml              # hub root (list → in-cluster → addons/hub)
    clusters/clusters-appset.yaml       # cluster inventory sync pattern
    addons/hub/                         # argo-cd + spoke-root (hub AppSets)
    addons/spoke/                       # day-2 AppSets (spoke reconciles)
    addons/oss/                         # deprecated
    workloads/                          # hub in-cluster IDP stack
  bootstrap/spoke/root-apps/            # Applications installed on spokes
  environments/
    default|dev|staging|prod/addons/    # shared / env Helm values
    clusters/<logical>/addons/          # per-cluster values (via addons_cluster_name)

idp-argocd-apps/
  appsets/namespace-as-service.yaml     # spoke-local; in-cluster destination
  env/<env>/<cluster>/<tenant>.yaml     # Backstage writes here
```

## Sync ownership

1. **Hub bootstrap** — `kubectl apply -f bootstrap/control-plane/exclude/bootstrap.yaml` creates Application `bootstrap-hub-addons` pointing at `addons/hub/`.
2. **Hub AppSets** — `addons-argocd` and `addons-spoke-root` select spokes with `enable_argocd=true` and use `destination.name`.
3. **Spoke roots** — Applications `spoke-addons` and `spoke-naas` land in spoke `argocd` namespace.
4. **Spoke reconcile** — agent/app-controller syncs spoke AppSets and NaaS with `destination.server: https://kubernetes.default.svc`.

## Spoke secret contract

Cluster secret on the spoke is usually named `in-cluster`.

| Kind | Keys | Example |
| --- | --- | --- |
| **Label** | `enable_argocd`, `enable_naas`, `enable_<addon>` | `enable_metrics_server=true` |
| **Annotation** | `addons_cluster_name`, `naas_cluster_name`, `addons_repo_*` | `addons_cluster_name=labs` |

Logical name (e.g. `labs`) must match:

- `environments/clusters/labs/addons/<chart>/values.yaml`
- `env/<env>/labs/<tenant>.yaml` in idp-argocd-apps
- Backstage catalog Resource `metadata.name` for that cluster

## Hub workloads vs spoke addons

| Concern | Location | Sync plane |
| --- | --- | --- |
| Keycloak / CNPG / ESO config / Vault bootstrap jobs | `bootstrap/control-plane/workloads/` | Hub in-cluster |
| metrics-server, envoy-gateway, optional platform charts | `addons/spoke/` + `environments/...` | Spoke |
| NaaS tenants | idp-argocd-apps | Spoke |

Some Helm AppSets still exist under `addons/spoke/` for charts that labs historically enabled via labels. Prefer hub `workloads/` for true hub-only IDP; use spoke AppSets for cluster-local platform addons.

## External references

- [Argo CD Agent architecture](https://argocd-agent.readthedocs.io/stable/concepts/architecture/)
- [GitOps Bridge template](https://github.com/gitops-bridge-dev/gitops-bridge-argocd-control-plane-template)
