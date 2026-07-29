# GitOps Control Plane (DevZero lab slim)

Based on [gitops-bridge-argocd-control-plane-template](https://github.com/gitops-bridge-dev/gitops-bridge-argocd-control-plane-template).

Folder structure and **cluster generators** match the upstream GitOps Bridge pattern. Addon ApplicationSets are limited to:

| ApplicationSet | Enable label | Chart |
| --- | --- | --- |
| `addons-argocd` | `enable_argocd=true` | argo-cd |
| `addons-metrics-server` | `enable_metrics_server=true` | metrics-server |
| `addons-envoy-gateway` | `enable_envoy_gateway=true` | gateway-helm (OCI) |
| `addons-gitea` | `enable_gitea=true` | gitea |

AWS addon AppSets under `bootstrap/control-plane/addons/aws/` are removed (directories kept). `clusters-appset` and `exclude/bootstrap.yaml` are unchanged from upstream.

## Repository layout

```text
├── bootstrap
│   └── control-plane
│       ├── addons
│       │   ├── aws/          # placeholder (.keep)
│       │   └── oss/          # metrics-server, envoy-gateway, gitea AppSets
│       ├── clusters
│       │   └── clusters-appset.yaml
│       └── exclude
│           └── bootstrap.yaml
└── environments
    ├── default/addons/       # shared Helm values
    ├── dev|staging|prod/addons/
    └── environments/clusters/<name>/addons/  (e.g. labs)
```

## Generators (unchanged pattern)

Each OSS ApplicationSet uses a **merge** generator:

1. `clusters` selector: `akuity.io/argo-cd-cluster-name NotIn [in-cluster]` **and** `enable_<addon>=true`
2. Optional version overrides for `environment: staging` / `environment: prod`

Value files resolve via cluster annotations/labels:

- `addons_repo_url` / `addons_repo_revision` / `addons_repo_basepath`
- `environments/default|{{environment}}/addons/<chart>/values.yaml`
- `clusters/{{name}}/addons/<chart>/values.yaml`

## Bootstrap

Apply upstream-style bootstrap (from `exclude/`, so it is not double-synced):

```bash
# Label/annotate spoke clusters (not the Argo CD management "in-cluster" when using Akuity convention)
# enable_metrics_server=true enable_envoy_gateway=true enable_gitea=true
# addons_repo_url=... addons_repo_revision=main addons_repo_basepath=

kubectl apply -f bootstrap/control-plane/exclude/bootstrap.yaml
```
