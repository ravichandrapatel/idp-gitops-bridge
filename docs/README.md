# FILE_NAME: README.md
# DESCRIPTION: Index for idp-gitops-bridge operational and architecture documentation
# VERSION: 1.1.0
# AUTHORS: ravichandrapatel

# Documentation index

Control-plane docs for the DevZero IDP GitOps Bridge (hub–spoke Argo CD Agents).

| Section | Path | Purpose |
| --- | --- | --- |
| **ADRs** | [adr/](adr/) | Decisions + artifacts (includes Vault; former architecture-review-decisions) |
| **Architecture** | [architecture/](architecture/) | Overview + Mermaid diagrams |
| **Runbooks** | [runbooks/](runbooks/) | Step-by-step operational procedures |
| **Day-2 operations** | [operations/](operations/) | Steady-state ops, ownership, SLOs |
| **Cluster notes** | [cluster/](cluster/) | Lab cluster fragments (e.g. CoreDNS) |

Legacy folder [architecture-review-decisions/](architecture-review-decisions/) redirects to [adr/](adr/).

## Start here

1. [ADR index](adr/README.md) — [0001 hub–spoke](adr/0001-hub-spoke-argocd-agents.md) · [0002 Vault Hub](adr/0002-vault-hub.md)
2. [Architecture overview](architecture/hub-spoke.md)
3. [Architecture diagrams](architecture/diagrams.md)
4. [Day-2 operations](operations/day-2.md)

Legacy entry point [hub-spoke-architecture.md](hub-spoke-architecture.md) redirects to the architecture section.
