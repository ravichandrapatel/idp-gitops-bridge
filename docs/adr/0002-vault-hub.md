# FILE_NAME: 0002-vault-hub.md
# DESCRIPTION: ADR — Vault Hub as centralized security broker (OSS multi-tenant)
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# ADR-0002: Vault Hub (centralized security broker)

| Field | Value |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-07-30 |
| **Deciders** | Platform / IDP (ravichandrapatel) |
| **Artifacts** | [vault/readme.md](vault/readme.md) (full spec), [vault/values.yaml](vault/values.yaml), [vault/local-transit-engine-values.yaml](vault/local-transit-engine-values.yaml) |
| **Related** | [ADR-0001](0001-hub-spoke-argocd-agents.md), hub `workloads/` + spoke/hub Vault addon values |

## Context

The IDP needs a single secrets and identity broker for multi-tenant Kubernetes (hub and spokes) without per-tenant Vault sprawl. Prior design work lived under `docs/architecture-review-decisions/vault/`; that material is now the artifact set for this ADR.

## Decision

1. Run a **Vault Hub** (OSS unless Enterprise is explicitly licensed) as the centralized security broker.
2. Enforce multi-tenancy via **path convention + policies** in OSS (Vault Namespaces / native DR replication remain Enterprise-only and labeled as such in the spec).
3. Keep the authoritative narrative and SOPs in [vault/readme.md](vault/readme.md); keep deployable Helm overlays next to the ADR under [vault/](vault/).
4. Retire the separate `docs/architecture-review-decisions/` tree as a content home — ADRs are the single decision index.

## Consequences

- Operators look up Vault under `docs/adr/` like every other control-plane decision.
- Values overlays version with the ADR artifacts; chart enablement still follows hub/spoke GitOps paths in `environments/` and AppSets.
- Full procedure detail stays in the Vault spec (not duplicated here).

## References

- [Vault Hub — Architectural Specification](vault/readme.md)
