# Config Path 02: Power Platform Custom Connector · Private Network · APIM · Databricks mcpsql

> **Scenario**: Copilot Studio agent uses a custom connector routed through APIM over a fully private Azure VNet path to query structured data in a Databricks SQL Warehouse via mcpsql.

## Applicable Pattern

→ [pattern-private-via-apim](../20-patterns/connectivity/pattern-private-via-apim.md)  
→ [pattern-dbx-mcp-sql](../20-patterns/databricks/pattern-dbx-mcp-sql.md)  
→ [pattern-oauth-client-credentials](../20-patterns/authentication/pattern-oauth-client-credentials.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Azure VNet provisioned in the same region as the Databricks workspace.
- [ ] Databricks workspace with a SQL Warehouse provisioned. See [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md).
- [ ] APIM instance provisioned (Developer SKU minimum). See [component-apim-private-ingress-egress](../10-components/apim/component-apim-private-ingress-egress.md).
- [ ] Entra ID app registration for OAuth 2.0 authentication. See [component-dbx-auth](../10-components/databricks/component-dbx-auth.md).
- [ ] Power Platform environment with custom connectors enabled.

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

### Step 4 — Deploy and Configure APIM

1. Follow [component-apim-private-ingress-egress](../10-components/apim/component-apim-private-ingress-egress.md) § "Configuration Steps" to deploy APIM in Internal VNet mode.
2. Follow [component-apim-mcp-proxy-basics](../10-components/apim/component-apim-mcp-proxy-basics.md) § "Configuration Steps" to create the MCP SQL API in APIM.
3. Test from the APIM test console — expect HTTP 200 from the mcpsql endpoint.

> **Note on authentication**: No additional APIM policy configuration is needed to handle Databricks authentication — it is passed through automatically.

### Step 5 — Deploy VNet Data Gateway

1. Follow [component-pp-networking](../10-components/power-platform/component-pp-networking.md) § "Private Network Path (VNet Data Gateway)".
2. Confirm the gateway is registered in Power Platform admin center and shows "Online".

### Step 6 — Build and Register Custom Connector

1. Follow [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md) § "Configuration Steps".
2. Set the connector's host URL to the APIM frontend URL.
3. Configure OAuth 2.0 client credentials for Power Platform → APIM authentication.
4. Set the connector to use the VNet data gateway from Step 5.
5. Test the connection — expect "Connected" status.

### Step 7 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action using the custom connector connection from Step 6.
3. Map user inputs to the MCP SQL query action. TODO: confirm action input schema.

### Step 8 — Validate End-to-End

1. Test the agent in Copilot Studio test chat with a data query.
2. Verify the expected SQL result is returned.
3. Check APIM diagnostics to confirm the request was proxied.
4. Check Databricks query history to confirm the SQL Warehouse executed the query.

## Validation Checklist

- [ ] Private Endpoint DNS resolves to a private IP from within the VNet.
- [ ] APIM test console returns HTTP 200 from mcpsql.
- [ ] VNet data gateway shows "Online" in Power Platform admin center.
- [ ] Custom connector connection status is "Connected".
- [ ] End-to-end Copilot Studio test returns expected SQL data.
- [ ] Databricks public network access can be disabled without breaking the integration.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| APIM cannot reach Databricks | DNS misconfiguration or Private Endpoint not ready | Verify DNS from APIM subnet; check Private Endpoint state |
| VNet data gateway offline | VM or gateway service stopped | Restart gateway; check VM health |
| Custom connector 404 | Wrong APIM URL or API suffix | Verify APIM URL and API path in connector definition |
