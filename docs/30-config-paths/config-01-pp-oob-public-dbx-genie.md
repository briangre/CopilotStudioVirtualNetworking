# Config Path 01: Power Platform OOB Connector · Public Network · Databricks mcpgenie

> **Scenario**: Copilot Studio agent uses the OOB Databricks connector over the public internet to answer natural-language questions via Databricks AI/BI Genie.

## Applicable Pattern

→ [pattern-public-direct](../20-patterns/connectivity/pattern-public-direct.md)  
→ [pattern-dbx-mcp-genie](../20-patterns/databricks/pattern-dbx-mcp-genie.md)

> **No separate auth pre-configuration required.** The OOB connector handles authentication (OAuth or API key) inline when you create the connection in Copilot Studio — you do not need to pre-register a service principal or configure credentials outside of the connector setup.

## Prerequisites

Confirm all prerequisites are met before starting:

- [ ] Databricks workspace provisioned with public network access **enabled**.
- [ ] AI/BI Genie Space created and a SQL Warehouse associated. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
- [ ] MCP server enabled for the Genie Space (see Step 1 below).
- [ ] Power Platform environment created (Copilot Studio enabled).
- [ ] Power Platform admin has verified the OOB Databricks connector is not blocked by DLP policy. See [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md).

## Steps

### Step 1 — Configure the Databricks Genie MCP Endpoint

1. In your Databricks workspace, navigate to **AI/BI > Genie** and open (or create) a Genie Space.
2. Enable the MCP server for the Genie Space: in the Genie Space settings, turn on **MCP Server**. Once enabled, the endpoint will appear automatically in **Agents > MCP Servers**.
3. The endpoint URL has the form `https://<workspace-host>/api/2.0/mcp/genie/<genie-space-id>` and is shown on the **Agents > MCP Servers** page.

> For full details on enabling and using Databricks managed MCP servers, see [Use Databricks managed MCP servers](https://learn.microsoft.com/en-us/azure/databricks/generative-ai/mcp/managed-mcp).

### Step 2 — Note the Databricks Hostname and SQL Warehouse HTTP Path

You will need two values from your Databricks workspace when creating the connector connection:

| Value | Where to find it |
|-------|-----------------|
| **Databricks workspace host name** | The hostname portion of your workspace URL, e.g. `adb-<workspace-id>.<region>.azuredatabricks.net`. Visible in the browser address bar when logged in to your workspace. |
| **SQL Warehouse HTTP path** | In your workspace: **SQL > SQL Warehouses** → select your warehouse → **Connection Details** tab → copy the **HTTP path** value, e.g. `/sql/1.0/warehouses/<warehouse-id>`. |

> For step-by-step instructions on finding these values, see [Get connection details for a Databricks compute resource](https://learn.microsoft.com/en-us/azure/databricks/integrations/compute-details).

### Step 3 — Add the Genie Tool to Your Copilot Studio Agent

1. In [Copilot Studio](https://copilotstudio.microsoft.com), open (or create) your agent.
2. Select **Tools** > **+ Add a tool** (in some Copilot Studio versions this appears as **Actions** > **+ Add an action**).
3. In the search box, type **azure databricks** to filter the connector gallery.
4. Filter the results on **MCP** and select the **Genie** action.
5. When prompted to create a connection:
   a. Select the **authentication type**: **OAuth** (recommended) or **API key**.
   b. Enter the **Databricks workspace host name** from Step 2.
   c. Enter the **SQL Warehouse HTTP path** from Step 2.
6. Select **Create connection** and complete any OAuth consent flow if prompted.
7. Confirm the connection status shows **Connected**.

### Step 4 — Validate End-to-End

1. Publish the agent and test in the Copilot Studio test chat.
2. Ask a sample question (e.g., "What are the top 10 products by revenue?").
3. Verify a meaningful answer is returned.
4. Check the Databricks AI/BI Genie Space query history to confirm the question was processed.

## Validation Checklist

- [ ] OOB connector connection status is "Connected".
- [ ] Copilot Studio agent tool returns a response from the MCP endpoint.
- [ ] Natural-language answer is displayed in the test chat.
- [ ] Databricks AI/BI Genie Space query history shows the question was received.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|-------------|------------|
| 401 on connector test | OAuth consent not completed or API key invalid | Re-create the connection and complete the OAuth consent flow, or verify the API key |
| DLP policy error | Connector blocked | Ask Power Platform admin to add the Azure Databricks connector to the allowed list |
| Timeout | Genie Space SQL Warehouse not running | Start the warehouse; check that auto-start is enabled |
| Empty or unexpected answer | Genie Space not configured correctly | Review Genie Space data assets and instructions |
| Cannot find "Azure Databricks" in tool search | Connector not visible | Confirm you are searching for **azure databricks** and filtering on **MCP** |
