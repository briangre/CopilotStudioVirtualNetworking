# Config Path 05: Power Platform Custom Connector · Public Network · Databricks mcpsql

> **Scenario**: Copilot Studio agent uses a custom connector over the public internet to query structured data in a Databricks SQL Warehouse via mcpsql. This is the quickest path to a working custom-connector prototype with no private networking infrastructure required.

## Applicable Pattern

→ [pattern-public-direct](../20-patterns/connectivity/pattern-public-direct.md)  
→ [pattern-dbx-mcp-sql](../20-patterns/databricks/pattern-dbx-mcp-sql.md)  
→ [pattern-oauth-client-credentials](../20-patterns/authentication/pattern-oauth-client-credentials.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Databricks workspace provisioned with public network access **enabled**.
- [ ] Databricks workspace with a SQL Warehouse provisioned. See [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md).
- [ ] Entra ID app registration created with Databricks API permission. See [component-dbx-auth](../10-components/databricks/component-dbx-auth.md).
- [ ] Power Platform environment with custom connectors enabled. See [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md).

## Steps

### Step 1 — Configure Databricks Authentication

1. Follow [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) § "Option 1: OAuth 2.0 – Client Credentials".
2. Note: `client_id`, `client_secret`, `tenant_id`, Databricks workspace URL, and SQL Warehouse HTTP Path.

### Step 2 — Configure the mcpsql Endpoint

1. Follow [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md) § "Configuration Steps".
2. Note: MCP SQL endpoint URL.
3. Confirm Databricks public network access is enabled and the workspace firewall permits inbound traffic from Power Platform IP ranges. TODO: confirm exact IP ranges.

### Step 3 — Build and Register Custom Connector

1. Follow [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md) § "Configuration Steps".
2. Set the connector's host URL directly to the Databricks workspace URL (no APIM in this path).
3. Configure OAuth 2.0 client credentials using the credentials from Step 1.
4. Test the connection — expect "Connected" status.

### Step 4 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action using the custom connector connection from Step 3.
3. Map user inputs to the MCP SQL query action. TODO: confirm action input schema.

### Step 5 — Validate End-to-End

1. Test the agent in Copilot Studio test chat with a data query.
2. Verify the expected SQL result is returned.
3. Check Databricks query history to confirm the SQL Warehouse executed the query.

## Validation Checklist

- [ ] Custom connector connection status is "Connected".
- [ ] Copilot Studio agent action returns HTTP 200 from the mcpsql endpoint.
- [ ] Expected SQL data is returned in the test chat.
- [ ] Databricks audit log shows the service principal's API call.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | Invalid token or wrong scope | Re-check client credentials and Databricks OAuth scope in [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) |
| Connection refused / timeout | Databricks public access disabled or firewall rule missing | Enable public network access; add Power Platform IP ranges to Databricks workspace firewall |
| DLP policy error | Custom connector host blocked | Ask Power Platform admin to update DLP policy to allow the Databricks endpoint |
| SQL Warehouse not responding | Warehouse stopped or starting up | Start the warehouse; check auto-start is enabled |
