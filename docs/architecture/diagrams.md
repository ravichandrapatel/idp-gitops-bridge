# FILE_NAME: diagrams.md
# DESCRIPTION: Mermaid architecture diagrams for hub–spoke GitOps
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Architecture diagrams

Companion to [hub-spoke.md](hub-spoke.md) and [ADR-0001](../adr/0001-hub-spoke-argocd-agents.md).

## 1. Control planes and Git

```mermaid
flowchart TB
  subgraph git [Git]
    Bridge["idp-gitops-bridge"]
    NaasGit["idp-argocd-apps"]
    Tpl["idp-backstage-templates"]
  end

  subgraph hub [Hub cluster]
    HubArgo[Hub_Argo_CD]
    Principal[argocd_agent_Principal]
    Workloads[hub_workloads]
    HubArgo --> Principal
    HubArgo -->|"bootstrap only"| Bridge
    HubArgo --> Workloads
  end

  subgraph spoke [Spoke cluster]
    Agent[Agent_autonomous]
    SpokeCtrl[Local_app_controller]
    Agent --> SpokeCtrl
    SpokeCtrl -->|"addons"| Bridge
    SpokeCtrl -->|"NaaS"| NaasGit
  end

  Backstage[Backstage] --> Tpl
  Backstage -->|"PR"| NaasGit
  Agent -->|"outbound_mTLS"| Principal
```

## 2. Hub bootstrap sequence

```mermaid
sequenceDiagram
  participant Ops as Operator
  participant Hub as Hub_Argo_CD
  participant Git as idp_gitops_bridge
  participant Spoke as Spoke_cluster

  Ops->>Hub: apply exclude/bootstrap.yaml
  Hub->>Git: sync addons/hub
  Hub->>Spoke: Application install Argo_CD enable_argocd
  Hub->>Spoke: Application sync spoke root-apps
  Note over Spoke: spoke-addons and spoke-naas created
  Spoke->>Git: sync addons/spoke
  Spoke->>Spoke: reconcile addon Apps in-cluster
```

## 3. NaaS request path (day-2)

```mermaid
flowchart LR
  User[Engineer] --> BS[Backstage_NaaS_template]
  BS -->|"PR auto-merge"| AppsRepo["idp-argocd-apps<br/>env/env/cluster/tenant.yaml"]
  AppsRepo --> AppSet[Spoke_NaaS_ApplicationSet]
  AppSet -->|"filter naas_cluster_name"| Helm["OCI chart<br/>namespace-as-service"]
  Helm --> NS["tenant namespaces<br/>in-cluster"]
```

## 4. Repo path ownership

```mermaid
flowchart LR
  subgraph hubPaths [Hub_syncs]
    B["exclude/bootstrap.yaml"]
    H["addons/hub/"]
    W["workloads/"]
  end

  subgraph spokePaths [Spoke_syncs]
    R["spoke/root-apps/"]
    S["addons/spoke/"]
    E["environments/clusters/logical/addons"]
    N["idp-argocd-apps env and appsets"]
  end

  B --> H
  H -->|"pushes roots"| R
  R --> S
  R --> N
  S --> E
```

## 5. Secret contract on spoke `in-cluster`

```mermaid
flowchart TB
  Secret["ClusterSecret_in-cluster"]
  Secret --> Labels["Labels<br/>enable_argocd<br/>enable_naas<br/>enable_addon"]
  Secret --> Ann["Annotations<br/>addons_cluster_name<br/>naas_cluster_name<br/>addons_repo_star"]
  Labels --> Gen[AppSet_generators]
  Ann --> Paths["values and git paths<br/>environments/clusters/logical<br/>env/env/logical"]
```

## Rendering

GitHub and most Markdown previews render Mermaid fenced blocks. For local preview, use any Mermaid-compatible viewer or the IDE Mermaid extension.
