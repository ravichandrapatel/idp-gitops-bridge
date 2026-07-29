# FILE_NAME: README.md
# DESCRIPTION: Index for idp-gitops-bridge operational and architecture documentation
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Documentation index

Control-plane docs for the DevZero IDP GitOps Bridge (hub–spoke Argo CD Agents).

| Section | Path | Purpose |
| --- | --- | --- |
| **ADRs** | [adr/](adr/) | Architecture Decision Records (binding choices) |
| **Architecture** | [architecture/](architecture/) | Overview + Mermaid diagrams |
| **Runbooks** | [runbooks/](runbooks/) | Step-by-step operational procedures |
| **Day-2 operations** | [operations/](operations/) | Steady-state ops, ownership, SLOs |
| **Vault decisions** | [architecture-review-decisions/](architecture-review-decisions/) | Vault hub values and design notes |
| **Cluster notes** | [cluster/](cluster/) | Lab cluster fragments (e.g. CoreDNS) |

## Start here

1. [ADR-0001: Hub–spoke Argo CD Agents](adr/0001-hub-spoke-argocd-agents.md)
2. [Architecture overview](architecture/hub-spoke.md)
3. [Architecture diagrams](architecture/diagrams.md)
4. [Day-2 operations](operations/day-2.md)

Legacy entry point [hub-spoke-architecture.md](hub-spoke-architecture.md) redirects to the architecture section.
