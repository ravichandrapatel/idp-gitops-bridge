# FILE_NAME: README.md
# DESCRIPTION: Index of operational runbooks for hub–spoke GitOps
# VERSION: 1.0.0
# AUTHORS: ravichandrapatel

# Runbooks

| Runbook | When to use |
| --- | --- |
| [bootstrap-hub.md](bootstrap-hub.md) | First-time or rebuild hub bootstrap ApplicationSet |
| [onboard-spoke.md](onboard-spoke.md) | Register a new spoke and install Argo CD + roots |
| [enable-addon.md](enable-addon.md) | Turn on/off a spoke platform addon |
| [provision-naas-tenant.md](provision-naas-tenant.md) | Tenant via Backstage / Git; verify spoke sync |
| [troubleshoot-sync.md](troubleshoot-sync.md) | OutOfSync, missing apps, wrong cluster path |

Related: [Day-2 operations](../operations/day-2.md) · [Architecture](../architecture/hub-spoke.md)
