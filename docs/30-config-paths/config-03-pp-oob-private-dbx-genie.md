# Config Path 03: Power Platform OOB Connector · Private Network · Databricks mcpgenie

> **Scenario**: Copilot Studio agent uses the OOB Databricks connector over a private Azure network path to answer natural-language questions via Databricks AI/BI Genie — without APIM.

## Applicable Pattern

→ [pattern-private-end-to-end](../20-patterns/connectivity/pattern-private-end-to-end.md)  
→ [pattern-dbx-mcp-genie](../20-patterns/databricks/pattern-dbx-mcp-genie.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Azure VNet provisioned in the same region as the Databricks workspace.
- [ ] Databricks workspace with AI/BI Genie Space and associated SQL Warehouse. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
- [ ] Power Platform environment with private network support (Premium license or pay-per-use). See [component-pp-networking](../10-components/power-platform/component-pp-networking.md).
- [ ] Power Platform admin has verified the OOB Databricks connector is not blocked by DLP policy. See [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md).

## Steps

### Step 1 — Configure the mcpgenie Endpoint

1. Follow [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md) § "Configuration Steps".
2. Note: MCP Genie endpoint URL and Genie Space ID.

### Step 2 — Deploy Private Endpoint for Databricks

1. Follow [component-public-vs-private](../10-components/networking/component-public-vs-private.md) § "Private Path" configuration steps.
2. Confirm DNS resolution of the Databricks hostname returns a private IP from within the VNet.

### Step 4 — Create the OOB Connector Connection

1. Follow [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md) § "Configuration Steps".
2. Supply the credentials from Step 1 and the endpoint URL from Step 2.
3. Test the connection — expect "Connected" status.

### Step 5 — Add Connector Action to Copilot Studio Agent

1. In Copilot Studio, open (or create) your agent.
2. Add an action and select the Databricks connector connection from Step 4.
3. Map the user's question to the `ask_question` action input. TODO: confirm action name.

### Step 6 — Validate End-to-End

1. Publish the agent and test in Copilot Studio test chat.
2. Ask a sample question.
3. Verify a meaningful natural-language answer is returned.
4. Check Databricks AI/BI Genie Space query history.
5. (Optional) Disable Databricks public network access and re-test to confirm private-only path.

## Validation Checklist

- [ ] Private Endpoint DNS resolves to a private IP from within the VNet.
- [ ] OOB connector connection status is "Connected".
- [ ] Copilot Studio agent action returns HTTP 200 from the MCP endpoint.
- [ ] Natural-language answer is displayed in the test chat.
- [ ] Databricks AI/BI Genie Space query history shows the question was processed.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | Invalid token or wrong scope | Re-check client credentials in [component-dbx-auth](../10-components/databricks/component-dbx-auth.md) |
| Connector cannot reach Databricks | DNS not resolving private IP | Verify DNS zone linkage; check network config |
| Timeout | Genie Space SQL Warehouse not running | Start the warehouse; check auto-start is enabled |
