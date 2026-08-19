# Roll-Up Checklist — From Beginner or Intermediate into Expert

Finished the 🟢 beginner or 🟡 intermediate track? You do **not** start over. Your deployment
becomes the expert track's starting point — provided it satisfies the contract the expert labs
assume.

For the shared validation evidence and known intermediate cleanup debt, see the
[beginner/intermediate parity report](../track-parity.md). That report is read-only evidence;
it does not authorize the expert-lab gate.

Work through this checklist with a coach. It takes about 15 minutes.

## 1. The workload is compliant

- [ ] Every [acceptance criterion](workload.md#acceptance-criteria) passes against the live deployment

If something fails, fix it agentically — that is still track work, and the expert labs will
only amplify the problem otherwise.

## 2. The deployment is reproducible

- [ ] The whole workload can be redeployed from source with a single command
- [ ] `./scripts/validate-infra.sh` passes
- [ ] `dotnet build src/ContosoTicketing` passes and the app is deployed to the App Service
- [ ] Every module in `infra/` uses a version-pinned Azure Verified Module
- [ ] A `.bicepparam` (or equivalent) holds all environment-specific values — no hardcoded ids in templates
- [ ] The database bootstrap (contained SQL user, `db_datareader`, `Tickets` table) runs automatically
  via the `Microsoft.Resources/deploymentScripts` module as part of `az deployment sub create` — no
  manual VM/Bastion step

Expert Lab 2 uses `what-if` as a drift detector; that only works if source is the source of truth.

## 3. The region supports the SRE Agent

- [ ] Resources are in `swedencentral`, `eastus2` or `australiaeast`

If not, redeploy into a supported region before Lab 3. Do it with your pipeline, not by hand — it is a good regression test.

## 4. Telemetry exists

- [ ] A Log Analytics workspace exists in the resource group
- [ ] Application Insights is workspace-based and receiving telemetry
- [ ] At least the web app and NSGs send diagnostics to the workspace
- [ ] `GET /healthz` returns `200` and `GET /readyz` returns `200` — the labs alert on both

The SRE Agent investigates using Azure Monitor data. No telemetry, no investigation.

## 5. Identity and permissions

- [ ] The web app uses a system-assigned managed identity
- [ ] No passwords or connection secrets in app settings, outputs or source
- [ ] Your account has `Contributor` **and** `User Access Administrator` on the subscription

Labs 3, 5 and 6 assign roles, enable Defender plans and create connectors. Labs 5 and 6 are optional.

## 6. Repository hygiene

- [ ] Your IaC lives in a GitHub repository you control
- [ ] The default branch is `main` and your work is merged into it
- [ ] Your architecture, plan, test results and runbook are committed

Labs 4–6 operate on the repository: PRs, CodeQL, branch protection and the Defender GitHub connector. Labs 5 and 6 (CodeQL, Defender connector) are optional.

## 7. Record your baseline

Note these — the expert labs reference them:

| Value | Yours |
|---|---|
| Subscription id | |
| Resource group name | |
| Region | |
| Web app name | |
| SQL server name | |
| Log Analytics workspace name | |
| Database bootstrap identity name | |
| GitHub repository | |
| Web app default host name | |

---

✅ All boxes ticked? Go to [🔴 expert track](../tracks/expert.md) and start at **Lab 1**.
