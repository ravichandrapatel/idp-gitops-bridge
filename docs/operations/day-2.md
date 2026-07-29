# FILE_NAME: day-2.md
# DESCRIPTION: Day-2 operations for hub–spoke Argo CD Agents
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Day-2 operations

Steady-state operating model after hub bootstrap and spoke onboarding.  
**ADR:** [ADR-0001](../adr/0001-hub-spoke-argocd-agents.md) · **Runbooks:** [../runbooks/](../runbooks/)

## 1. Ownership matrix

| Change type | Owner plane | Primary git | Procedure |
| --- | --- | --- | --- |
| Hub IDP workload (Keycloak realm, ESO, Vault bootstrap job) | Hub | `bootstrap/control-plane/workloads/` | PR → hub sync |
| Spoke Argo CD chart version / bootstrap | Hub AppSet | `addons/hub/` + `environments/**/argo-cd` | PR → hub; rolls spokes via `destination.name` |
| Spoke root App definition | Hub push / spoke pull | `bootstrap/spoke/root-apps/` | PR → hub `addons-spoke-root` refreshes spokes |
| Platform addon enable/values | Spoke | `addons/spoke/` + `environments/` | Label + values PR ([enable-addon](../runbooks/enable-addon.md)) |
| NaaS tenant create/update/delete | Spoke | idp-argocd-apps `env/...` | Backstage or Git PR |
| New spoke cluster | Hub then Spoke | cluster secret + catalog + values dir | [onboard-spoke](../runbooks/onboard-spoke.md) |
| Template / form UX | Portal | idp-backstage-templates | Tag pin in Backstage app-config |

## 2. Change management

### Freeze rules

- Do not introduce new hub ApplicationSets that set `destination.name` for day-2 addons or NaaS.
- Do not store tenant desired state in this repo; tenants belong in idp-argocd-apps.
- Pin production portal templates to git tags (e.g. `template-namespace-as-service-v1.1.1`).

### Promotion

1. Change values under `environments/default` → env overlay → `clusters/<logical>`.
2. Spokes track `addons_repo_revision` (usually `main` in lab; pin commit/tag in prod).
3. Chart bumps: update AppSet `addonChartVersion` in `addons/spoke/` (or hub argo-cd AppSet) via PR.

### Dual-control

- Hub operators manage bootstrap credentials and hub workloads.
- Spoke / platform operators manage `enable_*` labels and cluster annotations on spoke.
- Developers request NaaS via Backstage only (no direct cluster admin for namespace create).

## 3. Observability

| Signal | Where |
| --- | --- |
| Hub bootstrap health | Hub Argo: `bootstrap-hub-addons`, `addons-argocd`, `addons-spoke-root` |
| Spoke root health | Spoke Argo: `spoke-addons`, `spoke-naas` |
| Addon / tenant apps | Spoke Argo Applications |
| Agent session | Hub principal + spoke agent logs ([agent docs](https://argocd-agent.readthedocs.io/stable/concepts/architecture/)) |
| Backstage PR merge | idp-argocd-apps Actions `auto-merge-backstage` |

**Expectation:** Hub UI will not list every spoke addon/NaaS Application. That is intentional.

## 4. Operative SLOs (lab defaults)

| Item | Target |
| --- | --- |
| Hub bootstrap Application healthy | 99% over rolling 7d (lab) |
| Spoke root Apps healthy after onboard | Within 15m of Argo CD Ready |
| NaaS Application created after merge to main | Within 5m (git poll + sync) |
| Break-glass tenant YAML | Documented; use sparingly |

Adjust for production when promoting beyond labs.

## 5. Secrets and credentials

| Secret | Plane | Notes |
| --- | --- | --- |
| Spoke kubeconfig on hub | Hub | Required for bootstrap Applications only; minimize scope |
| Git / OCI pull creds | Spoke (+ hub as needed) | Repo secrets in Argo CD; OCI for NaaS chart |
| Vault / OIDC / DB | Hub workloads + ESO | See [architecture-review-decisions/vault](../architecture-review-decisions/vault/readme.md) |
| `in-cluster` annotations | Spoke | Non-secret path selectors; still treat repo URLs as config |

Never commit kubeconfigs, tokens, or `.env` files into this repository.

## 6. Backup and restore (GitOps-first)

Desired state is Git. Day-2 restore order:

1. Restore hub Argo + apply `exclude/bootstrap.yaml`.
2. Re-register spoke cluster secrets on hub (`enable_argocd`).
3. Wait for spoke Argo + root Apps.
4. Re-apply spoke `in-cluster` labels/annotations.
5. Confirm `spoke-addons` / `spoke-naas` recreate Applications from Git.

Cluster data (PVC) restore is out of band (Velero/storage); Git does not replace volume backups.

## 7. Decommission a spoke

1. Drain NaaS tenants (delete env files; confirm prune).
2. Remove addon `enable_*` labels; tear down stateful addons deliberately.
3. Remove `enable_argocd` on hub cluster secret; delete `spoke-root-*` and `addon-*-argo-cd`.
4. Unregister agent from principal; delete hub cluster secret.
5. Remove catalog Resource and empty `environments/clusters/<logical>/` if unused.

## 8. Escalation

| Class | First look | Escalate when |
| --- | --- | --- |
| Sync / AppSet | [troubleshoot-sync](../runbooks/troubleshoot-sync.md) | >30m or multi-spoke blast |
| Agent mTLS | Agent + principal logs | Certificate or registration failure |
| NaaS chart | Spoke app events + OCI | Chart regression across tenants |
| Hub IDP | `workloads/` + Vault ADR | Auth/login outage |

## 9. Related documents

- [Architecture](../architecture/hub-spoke.md)
- [Diagrams](../architecture/diagrams.md)
- [ADR-0001](../adr/0001-hub-spoke-argocd-agents.md)
- [Runbooks index](../runbooks/README.md)
