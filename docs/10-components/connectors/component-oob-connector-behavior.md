# Component: Out-of-Box (OOB) Connector Behavior

## Purpose

Describes the behavior, capabilities, and constraints of the pre-built (OOB) Power Platform connector for Azure Databricks (https://learn.microsoft.com/en-us/connectors/databricks/) and its MCP endpoints, as it exists in the connector gallery without customization.

## When to Use

Reference this component when:
- Evaluating whether an OOB connector is sufficient for your scenario.
- Understanding what the OOB connector sends and receives.

## Inputs / Prerequisites

- Power Platform environment with internet access (for public path) or configured VNets (for private path).
- Valid Databricks workspace URL.
- Connector enabled by the Power Platform admin in the tenant's Data Loss Prevention (DLP) policy.

> **No separate auth pre-configuration required.** Authentication (OAuth or API key) is configured inline when you create the connector connection. You do not need to pre-register a service principal or set up credentials before starting the connection wizard.

## Outputs / What "Done" Looks Like

- Connector action returns a structured response (JSON) from the Databricks MCP endpoint.
- No errors in the Copilot Studio Agent test.
- DLP policy does not block the connection.

## Connector Capabilities

The full set of actions exposed by the OOB connector is documented in the [Azure Databricks connector reference](https://learn.microsoft.com/en-us/connectors/databricks/).

- The **Genie** action specifically contains logic to communicate with the `mcpGenie` endpoint, enabling natural-language data queries via the Databricks Genie experience.
- All other actions within the connector call standard Databricks REST APIs that are exposed by the workspace (e.g., SQL statement execution, cluster management).
- Handles OAuth token acquisition and refresh automatically.

## Configuration Steps

1. In Copilot Studio, open (or create) your agent and select **Tools** (or **Actions**) > **+ Add a tool**.
2. In the search box, type **azure databricks** to filter the connector gallery.
3. Filter the results on **MCP** and select the **Genie** action.
4. When prompted to create a connection, select the **authentication type**:
   - **OAuth** – recommended; an OAuth consent flow will be initiated in the browser during connection creation.
   - **API key** – enter a Databricks personal access token.
5. Provide the following connection details:
   - **Azure Databricks workspace host name** – the hostname portion of your Databricks workspace URL (e.g., `adb-<workspace-id>.<region>.azuredatabricks.net`). Visible in the browser address bar when logged in to your workspace.
   - **SQL Warehouse HTTP path** – found in the Databricks UI under **SQL > SQL Warehouses** → select your warehouse → **Connection Details** tab (e.g., `/sql/1.0/warehouses/<warehouse-id>`). See [Get connection details for a Databricks compute resource](https://learn.microsoft.com/en-us/azure/databricks/integrations/compute-details).
6. Select **Create connection** and complete any OAuth consent flow if prompted.
7. Confirm the connection status is **Connected**.

## Validation Steps

1. Use the built-in "Test" feature in Copilot Studio to invoke a connector action.
2. Verify the response body matches the expected schema.
3. Check the Power Platform admin center for DLP policy violations.

## Known Limitations

- The OOB connector action set is fixed; you cannot add custom actions without building a custom connector.

## Related

- [component-custom-connector-auth](component-custom-connector-auth.md)
- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [decision-which-connector](../../40-decision-guides/decision-which-connector.md)
