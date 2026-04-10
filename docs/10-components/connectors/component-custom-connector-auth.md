# Component: Custom Connector Authentication

## Purpose

Describes how to add a Databricks MCP endpoint as a tool in Copilot Studio using the built-in **Model Context Protocol (MCP)** tool type. When you add an MCP tool, Copilot Studio automatically creates and manages a custom Power Platform connector under the covers — no manual OpenAPI import or connector authoring is required.

## When to Use

Reference this component when:
- You want to connect a Copilot Studio agent to a Databricks MCP endpoint (`mcpgenie` or `mcpsql`).
- You prefer Copilot Studio to manage the custom connector lifecycle automatically.
- The OOB connector does not support your required auth model or endpoint.

## Inputs / Prerequisites

- Copilot Studio agent (existing or new) in a Power Platform environment.
- Databricks MCP endpoint URL (e.g., `https://<workspace-url>/api/2.0/mcp/...`).
- Entra ID app registration with appropriate permissions and a client secret (for OAuth 2.0 client credentials flow).
- Tenant ID, Client ID, Client Secret, and OAuth token and refresh URLs for the app registration.

## Outputs / What "Done" Looks Like

- An MCP tool is added to the Copilot Studio agent.
- Copilot Studio has automatically created a custom connector and connection in the Power Platform environment.
- The agent can invoke Databricks MCP actions (e.g., run SQL, call Genie) via the MCP tool.

## Configuration Steps

### Step 1 — Open Your Agent in Copilot Studio

1. Navigate to [Copilot Studio](https://copilotstudio.microsoft.com) and open (or create) your agent.

### Step 2 — Add an MCP Tool

1. In the left navigation pane, select **Tools**.
2. Click **+ Add a tool**.
3. In the tool type picker, select **Model Context Protocol (MCP)**.

### Step 3 — Configure the MCP Server Connection

1. In the **Server URL** field, enter the Databricks MCP endpoint URL.
   - Example: `https://adb-1234567890123456.7.azuredatabricks.net/api/2.0/mcp/sql` for the mcpsql endpoint.
   - Replace the host with your actual Databricks workspace URL, visible in the browser when logged in to your workspace.
2. Give the tool a descriptive **Name** and optional **Description** that explains its purpose to the agent.

### Step 4 — Configure Authentication

1. Under **Authentication**, select **OAuth 2.0**.
2. Fill in the following fields:
   - **Token URL**: `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token`
   - **Client ID**: Client ID from the Entra ID app registration.
   - **Client Secret**: Client secret from the Entra ID app registration.
   - **Scope**: The Databricks OAuth scope. For Azure-hosted Databricks, use `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/user_impersonation` (this UUID is the well-known first-party Databricks application ID in Entra ID). Consult your Databricks admin if your deployment uses a different scope.
3. Click **Create** to proceed and create the connection. **NOTE:** Any errors here will indicate that the authentication settings may be incorrect.

### Step 5 — Complete and Publish

1. Review the tool summary — Copilot Studio will display the MCP Tools discovered from the MCP server.
2. Click **Save** to save the configuration.
3. Copilot Studio automatically creates a custom connector and connection in the background.
4. **Publish** the agent to make the MCP tool available.

## Validation Steps

1. In Copilot Studio, open the **Test** pane and send a query that should invoke the MCP tool.
2. Verify the agent calls the MCP tool and returns a valid response from Databricks.
3. In the Power Platform maker portal, navigate to **Data > Custom connectors** to confirm the auto-generated connector is present.
4. Confirm the connection shows **Connected** status under **Data > Connections**.

## Known Limitations

- The auto-generated custom connector is managed by Copilot Studio; manual edits to it may be overwritten.
- Custom connectors are environment-scoped; they must be re-created or imported into each environment.
- Secrets (client secret) are stored in the connector connection and are not rotated automatically.
- Custom connector definitions can be exported/imported as solution components for ALM.
- Maximum response payload size is limited by Power Platform connector limits.

## Related

- [component-oob-connector-behavior](component-oob-connector-behavior.md)
- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [component-apim-mcp-proxy-basics](../apim/component-apim-mcp-proxy-basics.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
