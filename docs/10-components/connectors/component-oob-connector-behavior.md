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
- Authentication configured as described in [component-pp-auth-models](../power-platform/component-pp-auth-models.md).
- Connector enabled by the Power Platform admin in the tenant's Data Loss Prevention (DLP) policy.

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

1. In Copilot Studio, add a new tool to your agent and search for the **Azure Databricks** connector.
2. Filter the displayed actions by typing **MCP** in the search/filter box to surface the **Genie** action.
3. Select the **Genie** action and choose **oAuth** as the authentication method (these docs cover oAuth only; API key is also available but not described here).
4. Provide the following connection details:
   - **Azure Databricks workspace host name** – the hostname portion of your Databricks workspace URL (e.g., `adb-<workspace-id>.<region>.azuredatabricks.net`).
   - **SQL Warehouse HTTP path** – found in the Databricks UI under **SQL Warehouses → your warehouse → Connection details → HTTP path** (e.g., `/sql/1.0/warehouses/<warehouse-id>`).
5. Complete the oAuth consent flow when prompted.
6. Confirm the connection status is **Connected**.

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
