# Config Path 04: Power Platform Custom Connector · Private Network · APIM · Databricks mcpgenie

> **Scenario**: Copilot Studio agent uses a custom connector routed through APIM over a fully private Azure VNet path to answer natural-language questions via Databricks AI/BI Genie.

## Applicable Pattern

→ [pattern-private-via-apim](../20-patterns/connectivity/pattern-private-via-apim.md)  
→ [pattern-dbx-mcp-genie](../20-patterns/databricks/pattern-dbx-mcp-genie.md)  
→ [pattern-oauth-client-credentials](../20-patterns/authentication/pattern-oauth-client-credentials.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Azure VNet provisioned in the same region as the Databricks workspace.
- [ ] Databricks workspace with AI/BI Genie Space and associated SQL Warehouse. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
- [ ] APIM instance provisioned (Developer SKU minimum). See [component-apim-private-ingress-egress](../10-components/apim/component-apim-private-ingress-egress.md).
- [ ] Entra ID app registration for OAuth 2.0 authentication. See [component-dbx-auth](../10-components/databricks/component-dbx-auth.md).
- [ ] Power Platform environment with custom connectors enabled.

## Steps

### Step 1 — Configure Databricks Authentication

1. Follow [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) § "Option 1: OAuth 2.0 – Client Credentials".
2. Note: `client_id`, `client_secret`, `tenant_id`, and Databricks workspace URL.

### Step 2 — Configure the mcpgenie Endpoint

1. Follow [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md) § "Configuration Steps".
2. Note: MCP Genie endpoint URL and Genie Space ID.

### Step 3 — Deploy Private Endpoint for Databricks

1. Follow [component-public-vs-private](../10-components/networking/component-public-vs-private.md) § "Private Path" configuration steps.
2. Confirm DNS resolution of the Databricks hostname returns a private IP from within the VNet.

### Step 4 — Deploy and Configure APIM

1. Follow [component-apim-private-ingress-egress](../10-components/apim/component-apim-private-ingress-egress.md) § "Configuration Steps" to deploy APIM in Internal VNet mode.
2. Follow [component-apim-mcp-proxy-basics](../10-components/apim/component-apim-mcp-proxy-basics.md) § "Configuration Steps" to create the MCP Genie API in APIM.
3. Test from the APIM test console — expect HTTP 200 from the mcpgenie endpoint.

> **Note on authentication**: No additional APIM policy configuration is needed to handle Databricks authentication — it is passed through automatically.

### Step 5 — Build and Register Custom Connector

1. Follow [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md) § "Configuration Steps".
2. Set the connector's host URL to the APIM frontend URL.
3. Configure OAuth 2.0 client credentials for Power Platform → APIM authentication.
4. Test the connection — expect "Connected" status.

### Step 6 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action using the custom connector connection from Step 5.
3. Map the user's question to the `ask_question` action input. TODO: confirm action name.

### Step 7 — Validate End-to-End

1. Publish the agent and test in Copilot Studio test chat.
2. Ask a sample question (e.g., "What are the top 10 products by revenue?").
3. Verify a meaningful natural-language answer is returned.
4. Check APIM diagnostics to confirm the request was proxied.
5. Check Databricks AI/BI Genie Space query history to confirm the question was processed.

## Validation Checklist

- [ ] Private Endpoint DNS resolves to a private IP from within the VNet.
- [ ] APIM test console returns HTTP 200 from mcpgenie.
- [ ] Custom connector connection status is "Connected".
- [ ] End-to-end Copilot Studio test returns a natural-language answer.
- [ ] Databricks public network access can be disabled without breaking the integration.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| APIM cannot reach Databricks | DNS misconfiguration or Private Endpoint not ready | Verify DNS from APIM subnet; check Private Endpoint state |
| Custom connector 404 | Wrong APIM URL or API suffix | Verify APIM URL and API path in connector definition |
| Timeout | Genie Space SQL Warehouse not running | Start the warehouse; check auto-start is enabled |
| Empty or unexpected answer | Genie Space not configured correctly | Review Genie Space data assets and instructions |
