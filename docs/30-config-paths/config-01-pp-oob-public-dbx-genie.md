# Config Path 01: Power Platform OOB Connector · Public Network · Databricks mcpgenie

> **Scenario**: Copilot Studio agent uses the OOB Databricks connector over the public internet to answer natural-language questions via Databricks AI/BI Genie.

## Applicable Pattern

→ [pattern-public-direct](../20-patterns/connectivity/pattern-public-direct.md)  
→ [pattern-dbx-mcp-genie](../20-patterns/databricks/pattern-dbx-mcp-genie.md)  
→ [pattern-oauth-client-credentials](../20-patterns/authentication/pattern-oauth-client-credentials.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Databricks workspace provisioned with public network access **enabled**.
- [ ] AI/BI Genie Space created and a SQL Warehouse associated. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
- [ ] Entra ID app registration created with Databricks API permission. See [component-dbx-auth](../10-components/databricks/component-dbx-auth.md).
- [ ] Power Platform environment created (Copilot Studio enabled).
- [ ] Power Platform admin has verified the OOB Databricks connector is not blocked by DLP policy. See [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md).

## Steps

### Step 1 — Configure Databricks Authentication

1. Follow [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) § "Option 1: OAuth 2.0 – Client Credentials".
2. Note: `client_id`, `client_secret`, `tenant_id`, and Databricks workspace URL.

### Step 2 — Configure the mcpgenie Endpoint

1. Follow [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md) § "Configuration Steps".
2. Note: MCP Genie endpoint URL and Genie Space ID.

### Step 3 — Create the OOB Connector Connection

1. Follow [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md) § "Configuration Steps".
2. Supply the credentials from Step 1 and the endpoint URL from Step 2.
3. Test the connection — expect "Connected" status.

### Step 4 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action and select the Databricks connector connection created in Step 3.
3. Choose the `ask_question` (or equivalent) action. TODO: confirm action name.
4. Map the user's question to the action input.

### Step 5 — Validate End-to-End

1. Publish the agent and test in the Copilot Studio test chat.
2. Ask a sample question (e.g., "What are the top 10 products by revenue?").
3. Verify a meaningful answer is returned.
4. Check the Databricks AI/BI Genie Space query history to confirm the question was processed.

## Validation Checklist

- [ ] OOB connector connection status is "Connected".
- [ ] Copilot Studio agent action returns HTTP 200 from the MCP endpoint.
- [ ] Natural-language answer is displayed in the test chat.
- [ ] Databricks audit log shows the service principal's API call.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | Invalid token or wrong scope | Re-check client credentials and Databricks OAuth scope in [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) |
| DLP policy error | Connector blocked | Ask Power Platform admin to add connector to allowed list |
| Timeout | Genie Space SQL Warehouse not running | Start the warehouse; check auto-start is enabled |
| Empty or unexpected answer | Genie Space not configured correctly | Review Genie Space data assets and instructions |
