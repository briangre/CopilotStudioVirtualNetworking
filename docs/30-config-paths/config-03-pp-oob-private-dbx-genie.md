# Config Path 03: Power Platform OOB Connector · Private Network · Databricks mcpgenie

> **Scenario**: Copilot Studio agent uses the OOB Databricks connector routed through a VNet data gateway over a private Azure network path to answer natural-language questions via Databricks AI/BI Genie — without APIM.

## Applicable Pattern

→ [pattern-private-end-to-end](../20-patterns/connectivity/pattern-private-end-to-end.md)  
→ [pattern-dbx-mcp-genie](../20-patterns/databricks/pattern-dbx-mcp-genie.md)

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Azure VNet provisioned in the same region as the Databricks workspace.
- [ ] Databricks workspace with AI/BI Genie Space and associated SQL Warehouse. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
- [ ] Power Platform environment with VNet data gateway support (Premium license or pay-per-use). See [component-pp-networking](../10-components/power-platform/component-pp-networking.md).
- [ ] Power Platform admin has verified the OOB Databricks connector is not blocked by DLP policy. See [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md).

> **Note**: This config path uses the OOB connector with a private network path. Confirm that the OOB connector supports VNet data gateway routing before proceeding. TODO: confirm OOB connector VNet data gateway support.

## Steps

### Step 1 — Configure the mcpgenie Endpoint

1. Follow [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md) § "Configuration Steps".
2. Note: MCP Genie endpoint URL and Genie Space ID.

### Step 2 — Deploy Private Endpoint for Databricks

1. Follow [component-public-vs-private](../10-components/networking/component-public-vs-private.md) § "Private Path" configuration steps.
2. Confirm DNS resolution of the Databricks hostname returns a private IP from within the VNet.

### Step 3 — Deploy VNet Data Gateway

1. Follow [component-pp-networking](../10-components/power-platform/component-pp-networking.md) § "Private Network Path (VNet Data Gateway)".
2. Deploy the VNet data gateway in the same VNet as the Databricks Private Endpoint.
3. Confirm the gateway is registered in Power Platform admin center and shows "Online".

### Step 4 — Create the OOB Connector Connection

1. Follow [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md) § "Configuration Steps".
2. Supply the endpoint URL from Step 1.
3. Configure the connection to use the VNet data gateway from Step 3.
4. Test the connection — expect "Connected" status.

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
- [ ] VNet data gateway shows "Online" in Power Platform admin center.
- [ ] OOB connector connection status is "Connected".
- [ ] Copilot Studio agent action returns HTTP 200 from the MCP endpoint.
- [ ] Natural-language answer is displayed in the test chat.
- [ ] Databricks AI/BI Genie Space query history shows the question was processed.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | OAuth consent not completed | Re-run the OAuth consent flow in the connector connection setup |
| Connector cannot reach Databricks | DNS not resolving private IP from gateway | Verify DNS zone linkage; check gateway VM network config |
| VNet data gateway offline | VM or gateway service stopped | Restart gateway; check VM health |
| OOB connector ignores VNet gateway | OOB connector may not support VNet gateway routing | Switch to custom connector ([config-02](config-02-pp-custom-private-apim-dbx-sql.md)) or confirm with Microsoft support |
| Timeout | Genie Space SQL Warehouse not running | Start the warehouse; check auto-start is enabled |
