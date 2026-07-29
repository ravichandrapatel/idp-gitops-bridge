# FILE_NAME: 0001-hub-spoke-argocd-agents.md
# DESCRIPTION: ADR — hub bootstrap + spoke Argo CD Agent autonomous day-2
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# ADR-0001: Hub–spoke Argo CD Agents (autonomous)

| Field | Value |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-07-30 |
| **Deciders** | Platform / IDP (ravichandrapatel) |
| **Repos** | [idp-gitops-bridge](https://github.com/ravichandrapatel/idp-gitops-bridge), [idp-argocd-apps](https://github.com/ravichandrapatel/idp-argocd-apps), [idp-backstage-templates](https://github.com/ravichandrapatel/idp-backstage-templates) |
| **Related** | [Architecture](../architecture/hub-spoke.md), [Day-2 ops](../operations/day-2.md) |

## Context

The control plane started as classic **GitOps Bridge hub-push**: ApplicationSets on the hub used cluster generators and `destination.name` to deploy addons (and NaaS) onto every labeled spoke. That pattern:

- Concentrates spoke kubeconfig / cluster secrets on the hub for day-2 workloads
- Couples addon blast radius and sync load to a single Argo CD
- Conflicts with the desired boundary: hub owns **cluster lifecycle + spoke bootstrap**; spokes own **namespaces, tenants, and platform addons**

We already had [idp-gitops-bridge](https://github.com/ravichandrapatel/idp-gitops-bridge) as the GitOps Bridge control-plane repo. Inventing a parallel “cluster-gitops” repository would duplicate generators, values layout, and operational knowledge.

NaaS tenant desired state lives in [idp-argocd-apps](https://github.com/ravichandrapatel/idp-argocd-apps) at `env/<env>/<cluster>/<tenant>.yaml`, written by Backstage.

## Decision

1. **Evolve `idp-gitops-bridge`** — do not create a second control-plane git repo.
2. **Hub Argo CD** owns only:
   - Cluster inventory / registration patterns (`clusters` AppSet, bootstrap)
   - One-time **spoke Argo CD** (or Argo CD Agent runtime) install via hub AppSet (`destination.name`)
   - Thin **spoke root Applications** push (`bootstrap/spoke/root-apps/`)
   - Hub **in-cluster** IDP workloads under `bootstrap/control-plane/workloads/`
3. **Spoke Argo CD Agent in autonomous mode** owns day-2:
   - Addon ApplicationSets under `bootstrap/control-plane/addons/spoke/` with `destination.server: https://kubernetes.default.svc`
   - NaaS ApplicationSet from `idp-argocd-apps` with the same in-cluster destination
4. **Logical cluster name** on spoke secrets that are literally named `in-cluster` is carried by annotations:
   - `addons_cluster_name` → `environments/clusters/<name>/addons/`
   - `naas_cluster_name` → `env/<env>/<name>/*.yaml`
5. Enable flags remain **labels** (`enable_argocd`, `enable_naas`, `enable_<addon>`).

## Alternatives considered

| Option | Why not |
| --- | --- |
| Keep full hub-push GitOps Bridge | Hub retains day-2 credentials and sync for all addons/NaaS; does not meet scale/security goal |
| New dedicated cluster-gitops repo | Duplicates bridge layout; two sources of truth for addon values |
| Spoke managed mode only (hub still owns Applications) | Hub still authoritatively syncs spoke apps; weaker autonomy |
| Per-tenant hub Application with `destination.name` | Same remote-push model NaaS was escaping |

## Consequences

### Positive

- Clear ownership: hub bootstrap vs spoke reconcile
- Spoke can sync when hub is degraded (autonomous agent + local app-controller)
- NaaS path shape for Backstage stays `env/<env>/<cluster>/<tenant>.yaml`
- Addon Helm values stay in familiar `environments/` tree

### Negative / costs

- Migration: prune obsolete hub Applications that targeted `addons/oss/`
- Operators must label **and** annotate spoke `in-cluster` secrets correctly
- Hub UI no longer shows full day-2 app health for spoke addons (use spoke Argo / agent principal observability)
- Hub still needs cluster credentials for **bootstrap** (Argo CD + root Apps), until/unless bootstrap is also agent-delivered

### Compliance with repo layout

```text
bootstrap/control-plane/addons/hub/     # hub-only AppSets
bootstrap/control-plane/addons/spoke/   # spoke-reconciled AppSets
bootstrap/spoke/root-apps/              # spoke-addons + spoke-naas
bootstrap/control-plane/addons/oss/     # deprecated pointer
```

## Follow-ups

- Principal / agent mTLS and registration runbook hardening
- Optional: retire hub cluster secrets used only for day-2 (keep bootstrap-scoped secrets)
- Dual-run period checklist before deleting legacy hub-pushed Applications

## References

- [Argo CD Agent — architecture](https://argocd-agent.readthedocs.io/stable/concepts/architecture/)
- [GitOps Bridge control-plane template](https://github.com/gitops-bridge-dev/gitops-bridge-argocd-control-plane-template)
