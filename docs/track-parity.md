# Beginner/intermediate parity report

**Evidence date:** 19-08-2026  
**Subscription:** `43f0ee2a-7007-4813-8a68-0010cae6a9c3`  
**Tenant:** `00a492bc-c7b0-4a44-a31b-d4a583064768`  
**Region:** `swedencentral`

This is a read-only comparison of the designated beginner, intermediate and expert
resource groups before Expert Lab 1. No expert lab was run and no Azure resource was
created, changed or deleted.

## Conclusion

The designated beginner and intermediate deployments are **materially equivalent to
the expert starting topology and effective runtime configuration**, but the expert
lab gate is **not yet safe to authorize** from this evidence alone:

- The intermediate resource group contains a separate legacy `ticketing` deployment
  alongside the designated `ticketingint` deployment. It is cleanup debt, not drift
  in the designated workload.
- Azure's resource inventory reports missing required tags on provider-generated
  children in every group. This conflicts with the workload document's literal
  "every resource" acceptance criterion and needs an explicit owner/exception or
  remediation decision before the roll-up checklist can be marked complete.
- The application routes, private SQL path, managed identities and bootstrap all
  passed the live checks listed below.

Keep `expert-lab-gate` pending until the tag finding and intermediate cleanup debt
are dispositioned by the owner.

## Parity matrix

| Area | Beginner | Intermediate designated `ticketingint` | Expert starting point | Result |
|---|---|---|---|---|
| Resource topology | 34 ARM resources; one baseline stack | 33 baseline resources plus 29 legacy `ticketing` resources | 33 ARM resources; one baseline stack | Baseline equivalent; intermediate has cleanup debt |
| Region and network | Sweden Central; `10.10.0.0/16`; app, private-endpoint and deployment-script subnets | Same topology and prefixes | Same topology and prefixes | Equivalent |
| App Service | Linux P0v3, .NET 8, system-assigned identity, HTTPS-only, TLS 1.2, FTPS disabled, `/healthz` health check | Same | Same | Equivalent |
| SQL | Serverless General Purpose `GP_S_Gen5`, public access disabled, Entra-only authentication | Same | Same | Equivalent |
| Private SQL path | Approved SQL private endpoint and `privatelink.database.windows.net` VNet link | Same | Same | Equivalent |
| NSGs | Three subnet NSGs; each has inbound deny priority 4096 | Same; PEP NSG also has the expected allow rule | Same | Equivalent |
| Bootstrap | User-assigned identity is the sole Entra SQL administrator; AzureCLI deployment script succeeded with `OnExpiration`/`P1D` | Same | Same | Equivalent |
| Monitoring | Workspace-based App Insights, readiness web test and baseline alerts | Same for designated stack | Same | Equivalent |
| Application | `/healthz`, `/readyz`, `/api/tickets` returned HTTP 200 | Same | Same | Equivalent |
| Required tags | 10 provider-generated resources missing one or more required tags | 18 total in RG, including legacy resources; designated stack also has generated-resource gaps | 9 provider-generated resources missing one or more required tags | Blocker against literal acceptance criterion |

Names intentionally differ by workload token (`ticketing`, `ticketingint`,
`ticketingexp`) and resource group. Generated storage, container-group, network
interface, alert and Event Grid child names are also expected to differ.

## Intermediate cleanup debt

The intermediate RG contains an older `ticketing` stack in addition to the
designated `ticketingint` stack. The legacy set includes an older App Service,
SQL server/database, VNet, private endpoint, workspace/App Insights, bootstrap
identity/script, storage/container-group children, web test and alert resources.
Do not use those resources as evidence for the intermediate roll-up; remove or
formally exclude them through the resource owner after the lab exercise.

## Documentation and source alignment

- `docs/concepts/workload.md`, the three track guides, `docs/desired-state.md` and
  `docs/operations-runbook.md` describe the same private SQL, managed-identity,
  workspace-based telemetry and deployment-script bootstrap pattern observed live.
- PR [#41](https://github.com/bram-boer/AgenticEngineering-Hackathon-Infrastructure/pull/41)
  is **merged into `main`**. The current source and live deployments therefore
  align on the user-assigned bootstrap identity fix; the App Service identity
  remains system-assigned.
- The README's euro amounts are explicitly planning ranges, not quotes. A
  read-only retail-price lookup for P0v3 and `GP_S_Gen5` returned no rows, so the
  ranges were not independently reproduced here. Teams must recalculate them in
  the linked Azure Pricing Calculator for their agreement, region and telemetry
  volume.

## Read-only evidence collected

The comparison used Azure subscription/resource-group inventory, ARM resource
listing, App Service and App Service Plan reads, SQL server/database and private
endpoint reads, VNet/subnet/private DNS reads, NSG rule reads, deployment-script
reads, App Insights reads, and HTTPS GETs to the three application routes. Secret
values were not printed; only setting names and non-secret shape checks were
recorded. No deployment, update, delete, lab action or resource mutation was
performed.
