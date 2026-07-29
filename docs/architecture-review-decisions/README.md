# Architecture review decisions

Design notes and values for the DevZero IDP GitOps control plane.

Control-plane ADRs and hub–spoke ops docs: **[../README.md](../README.md)** · **[ADR-0001](../adr/0001-hub-spoke-argocd-agents.md)**.

## Vault

| File | Purpose |
| --- | --- |
| [vault/readme.md](vault/readme.md) | Vault Hub architectural specification (incl. transit / multi-tenant guidance) |
| [vault/values.yaml](vault/values.yaml) | Primary Helm values for the Vault addon |
| [vault/local-transit-engine-values.yaml](vault/local-transit-engine-values.yaml) | Local transit-engine values overlay |
