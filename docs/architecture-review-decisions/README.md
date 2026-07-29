# Architecture review decisions

Captured design notes and values from related platform repos, kept here for the DevZero IDP GitOps control plane.

## Vault

Source: [`ravichandrapatel/platform-apps-of-apps` → `addons/vault`](https://github.com/ravichandrapatel/platform-apps-of-apps/tree/main/addons/vault)

| File | Purpose |
| --- | --- |
| [vault/readme.md](vault/readme.md) | Vault Hub architectural specification (incl. transit / multi-tenant guidance) |
| [vault/values.yaml](vault/values.yaml) | Primary Helm values for the Vault addon |
| [vault/local-transit-engine-values.yaml](vault/local-transit-engine-values.yaml) | Local transit-engine values overlay (upstream placeholder was empty; stub retained) |
