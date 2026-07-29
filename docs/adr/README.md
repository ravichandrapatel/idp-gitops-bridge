# FILE_NAME: README.md
# DESCRIPTION: Unified Architecture Decision Records (ADRs) and decision artifacts
# VERSION: 2.0.0
# AUTHORS: ravichandrapatel

# Architecture Decision Records

Single home for **binding decisions** and their supporting artifacts (specs, values overlays).

Status values: `Proposed` · `Accepted` · `Superseded` · `Deprecated`.

| ID | Title | Status | Artifacts |
| --- | --- | --- | --- |
| [0001](0001-hub-spoke-argocd-agents.md) | Hub–spoke Argo CD Agents (autonomous) | Accepted | — |
| [0002](0002-vault-hub.md) | Vault Hub (centralized security broker) | Accepted | [vault/](vault/) |

## Layout

```text
docs/adr/
  README.md
  0001-hub-spoke-argocd-agents.md
  0002-vault-hub.md
  vault/                    # ADR-0002 artifacts (spec + Helm values)
    readme.md
    values.yaml
    local-transit-engine-values.yaml
```

## Template

1. Add `NNNN-short-title.md` using the structure of `0001-*.md` or `0002-*.md`.
2. Set status to `Proposed` until reviewed; then `Accepted`.
3. Put large specs/values under `docs/adr/<topic>/` and link them from the ADR.
4. Add a row to the table above.
