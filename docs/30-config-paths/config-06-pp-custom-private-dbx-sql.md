# Config Path 06: Power Platform Custom Connector · Private Network · Databricks mcpsql

> **Scenario**: Copilot Studio agent uses a custom connector routed through a VNet data gateway over a fully private Azure network path to query structured data in a Databricks SQL Warehouse via mcpsql — without APIM.

## Applicable Pattern

→ [pattern-private-end-to-end](../20-patterns/connectivity/pattern-private-end-to-end.md)  
→ [pattern-dbx-mcp-sql](../20-patterns/databricks/pattern-dbx-mcp-sql.md)  
→ [pattern-oauth-client-credentials](../20-patterns/authentication/pattern-oauth-client-credentials.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Azure VNet provisioned in the same region as the Databricks workspace.
- [ ] Databricks workspace with a SQL Warehouse provisioned. See [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md).
- [ ] Entra ID app registration created with Databricks API permission. See [component-dbx-auth](../10-components/databricks/component-dbx-auth.md).
- [ ] Power Platform environment with VNet data gateway support (Premium license or pay-per-use). See [component-pp-networking](../10-components/power-platform/component-pp-networking.md).
- [ ] Power Platform environment with custom connectors enabled. See [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md).

## Steps

### Step 1 — Configure Databricks Authentication

1. Follow [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) § "Option 1: OAuth 2.0 – Client Credentials".
2. Note: `client_id`, `client_secret`, `tenant_id`, Databricks workspace URL, and SQL Warehouse HTTP Path.

### Step 2 — Configure the mcpsql Endpoint

1. Follow [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md) § "Configuration Steps".
2. Note: MCP SQL endpoint URL.

### Step 3 — Deploy Private Endpoint for Databricks

1. Follow [component-public-vs-private](../10-components/networking/component-public-vs-private.md) § "Private Path" configuration steps.
2. Confirm DNS resolution of the Databricks hostname returns a private IP from within the VNet.

### Step 4 — Deploy VNet Data Gateway

1. Follow [component-pp-networking](../10-components/power-platform/component-pp-networking.md) § "Private Network Path (VNet Data Gateway)".
2. Deploy the VNet data gateway in the same VNet as the Databricks Private Endpoint.
3. Confirm the gateway is registered in Power Platform admin center and shows "Online".

### Step 5 — Build and Register Custom Connector

1. Follow [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md) § "Configuration Steps".
2. Set the connector's host URL directly to the Databricks workspace URL (no APIM in this path).
3. Configure OAuth 2.0 client credentials using the credentials from Step 1.
4. Set the connector to use the VNet data gateway from Step 4.
5. Test the connection — expect "Connected" status.

### Step 6 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action using the custom connector connection from Step 5.
3. Map user inputs to the MCP SQL query action. TODO: confirm action input schema.

### Step 7 — Validate End-to-End

1. Test the agent in Copilot Studio test chat with a data query.
2. Verify the expected SQL result is returned.
3. Check Databricks query history to confirm the SQL Warehouse executed the query.
4. (Optional) Disable Databricks public network access and re-test to confirm private-only path.

## Validation Checklist

- [ ] Private Endpoint DNS resolves to a private IP from within the VNet.
- [ ] VNet data gateway shows "Online" in Power Platform admin center.
- [ ] Custom connector connection status is "Connected".
- [ ] Copilot Studio agent action returns HTTP 200 from the mcpsql endpoint.
- [ ] Expected SQL data is returned in the test chat.
- [ ] Databricks public network access can be disabled without breaking the integration.
- [ ] Databricks audit log shows the service principal's API call.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | Invalid token or wrong scope | Re-check client credentials in [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) |
| Connector cannot reach Databricks | DNS not resolving private IP from gateway | Verify DNS zone linkage; check gateway VM network config |
| VNet data gateway offline | VM or gateway service stopped | Restart gateway; check VM health |
| Custom connector 404 | Wrong host URL or API path in connector definition | Verify Databricks workspace URL and mcpsql path in connector definition |
| SQL Warehouse not responding | Warehouse stopped or starting up | Start the warehouse; check auto-start is enabled |
