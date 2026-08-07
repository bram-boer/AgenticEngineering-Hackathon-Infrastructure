# Contoso Ticketing Operations Runbook

This runbook covers the reference deployment in `infra/`. Source control is the desired state; make
persistent changes through Bicep and, after completing
[Expert Lab 2](expert/lab-02-desired-state.md), use your `docs/desired-state.md` checks after every
deployment or incident.

## Deployment context

The operational scripts read resource names from the outputs of the **subscription deployment**.
Use the same deployment name that was passed to `az deployment sub create --name`:

```bash
deployment_name=ticketing-baseline
az deployment sub show \
  --name "$deployment_name" \
  --query "{state:properties.provisioningState, outputs:properties.outputs}" \
  -o json
```

If the name is unknown, list recent deployments rather than guessing:

```bash
az deployment sub list \
  --query "sort_by([].{name:name, timestamp:properties.timestamp, state:properties.provisioningState}, &timestamp)[-10:]" \
  -o table
```

## Bootstrap SQL access

The Bicep deployment creates the SQL server, database, private endpoint, private DNS, and App
Service managed identity. Azure Resource Manager cannot create the contained database user. The
App Service identity also **cannot create its own SQL principal**: until an Entra SQL administrator
creates that principal, the identity has no database authorization with which to execute `CREATE
USER`. This bootstrap is a one-time data-plane trust establishment and must be performed by the
configured Microsoft Entra SQL administrator.

Run the idempotent script from a host that is connected to the workload VNet, such as a secured
management VM, a self-hosted runner, or a workstation connected through approved private
connectivity:

```bash
az login
az account show --query "{subscription:name, user:user.name, tenant:tenantId}" -o table

./scripts/bootstrap-ticketing-database.sh \
  --deployment-name ticketing-baseline
```

The script:

1. Reads the resource group, app, SQL server, and database from deployment outputs.
2. Refuses to continue unless the SQL hostname resolves to an RFC 1918 private address.
3. Uses `sqlcmd -G` and the signed-in Entra SQL administrator; it uses no SQL password.
4. Idempotently creates the App Service identity as a contained user, grants `db_datareader`, and
   creates `dbo.Tickets`.

Do not temporarily enable SQL public access. Do not put an administrator token, password, SQL
connection secret, or generated access token in a workflow or repository variable.

Verify the bootstrap from the same private host:

```bash
web_app_host="$(az deployment sub show --name ticketing-baseline \
  --query properties.outputs.webAppHostName.value -o tsv)"

curl --fail-with-body "https://${web_app_host}/healthz"
curl --fail-with-body "https://${web_app_host}/readyz"
curl --fail-with-body "https://${web_app_host}/api/tickets"
```

Expected results are HTTP `200` for all three routes. A `503` from `/readyz` or `500` from
`/api/tickets` after bootstrap should be investigated with
[`knowledge/runbook-http-500.md`](../knowledge/runbook-http-500.md).

## Alert triage

The deployment defines alerts for App Service HTTP 5xx responses, response time, health-check
status, `/readyz` availability, and SQL CPU. Alert rules intentionally have no notification action
group because notification destinations are environment-specific. Add an approved action group in
the consuming environment or connect the rules to an Azure SRE Agent scoped to the workload
resource group.

Never inject a lab fault into Bicep or `main`. Expert Lab 4 faults are temporary, manually recorded
drift and must be reconciled from source after the exercise.
