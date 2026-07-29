# GitOps Control Plane (DevZero lab slim)

Based on [gitops-bridge-argocd-control-plane-template](https://github.com/gitops-bridge-dev/gitops-bridge-argocd-control-plane-template).

## Hub–spoke Argo CD Agents

This repo is split for **hub bootstrap** vs **spoke-owned** reconciliation:

| Zone | Path | Who syncs |
| --- | --- | --- |
| Hub AppSets | `bootstrap/control-plane/addons/hub/` | Hub Argo CD (in-cluster) |
| Spoke AppSets | `bootstrap/control-plane/addons/spoke/` | Spoke Argo CD Agent (autonomous, in-cluster) |
| Spoke roots | `bootstrap/spoke/root-apps/` | Hub pushes onto spokes; spoke then reconciles |
| Deprecated | `bootstrap/control-plane/addons/oss/` | Moved → see `hub/` and `spoke/` |

**Hub workloads** (Keycloak, Vault bootstrap, ESO config, etc.) stay under `bootstrap/control-plane/workloads/` and remain hub / in-cluster concerns.

Full diagram and migration: **[docs/hub-spoke-architecture.md](docs/hub-spoke-architecture.md)**.

Addon ApplicationSets (labels / charts):

| ApplicationSet | Enable label | Owner |
| --- | --- | --- |
| `addons-argocd` | `enable_argocd=true` | **Hub** (pushes to spokes) |
| `addons-spoke-root` | `enable_argocd=true` | **Hub** (pushes root Apps) |
| `addons-metrics-server` | `enable_metrics_server=true` | **Spoke** |
| `addons-envoy-gateway` | `enable_envoy_gateway=true` | **Spoke** |
| `addons-gitea` | `enable_gitea=true` | **Spoke** |
| `addons-backstage`, `vault`, `keycloak`, `tekton`, `external-secrets`, `cloudnative-pg` | matching `enable_*` | **Spoke** |

AWS addon AppSets under `bootstrap/control-plane/addons/aws/` are removed (directories kept).

## Repository layout

```text
├── bootstrap
│   ├── control-plane
│   │   ├── addons
│   │   │   ├── aws/          # placeholder (.keep)
│   │   │   ├── hub/          # argo-cd + spoke-root AppSets (hub-only)
│   │   │   ├── spoke/        # addon AppSets (spoke Argo CD, in-cluster)
│   │   │   └── oss/          # deprecated → hub/ + spoke/
│   │   ├── clusters
│   │   │   └── clusters-appset.yaml
│   │   ├── exclude
│   │   │   └── bootstrap.yaml   # hub-only list → addons/hub
│   │   └── workloads/        # hub in-cluster workloads
│   └── spoke
│       └── root-apps/        # spoke-addons + spoke-naas Applications
├── docs
│   └── hub-spoke-architecture.md
└── environments
    ├── default/addons/
    ├── dev|staging|prod/addons/
    └── clusters/<name>/addons/
```

## Generators

**Hub** `addons-argo-cd` still uses a **merge** generator and `destination.name` so the hub can install Argo CD on registered spokes.

**Spoke** addon AppSets use in-cluster `destination.server` and are loaded by the spoke root Application `spoke-addons`. Value files still resolve via cluster annotations/labels when generators select clusters:

- `addons_repo_url` / `addons_repo_revision` / `addons_repo_basepath`
- `environments/default|{{environment}}/addons/<chart>/values.yaml`
- `clusters/{{metadata.annotations.addons_cluster_name}}/addons/<chart>/values.yaml` (logical name; secret is often `in-cluster`)

## Bootstrap (hub only)

```bash
# Apply on the hub Argo CD cluster (not double-synced from exclude/)
kubectl apply -f bootstrap/control-plane/exclude/bootstrap.yaml

# Spokes: enable_argocd=true (+ addon enable_* labels as needed)
# Spoke in-cluster secret: enable_* labels; addons_cluster_name + naas_cluster_name annotations
```

Bootstrap uses a **list** generator targeting `https://kubernetes.default.svc` and repo
`https://github.com/ravichandrapatel/idp-gitops-bridge.git` (annotation-based overrides can be
reintroduced later if needed).
