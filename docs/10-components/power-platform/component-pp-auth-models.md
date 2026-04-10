# Component: Power Platform Authentication Models

## Purpose

Describes the authentication models supported by Power Platform connectors when calling external APIs (including Databricks MCP endpoints), and how to configure them.

## When to Use

Reference this component when:
- Choosing an authentication scheme for a new connector connection.
- Troubleshooting 401/403 errors from Power Platform to Databricks.
- Understanding the trade-offs between application identity (OAuth) and API key authentication.

## Inputs / Prerequisites

- Entra ID (Azure AD) tenant with permission to register app registrations and grant admin consent.
- Databricks workspace configured to accept OAuth tokens from the Entra ID tenant.
- Client ID, client secret, and tenant ID for the app registration.

## Outputs / What "Done" Looks Like

- Connector connection successfully authenticates and receives a valid access token.
- Databricks API calls return HTTP 200 (not 401/403).
- Token expiry and refresh behave correctly without manual intervention.

## Supported Auth Models

Three authentication options are available when connecting to Databricks. **OAuth via an Entra ID app registration with On-Behalf-Of (OBO) is the recommended approach for Copilot Studio agents using MCP Tools.** See [component-dbx-auth](../databricks/component-dbx-auth.md) for a full comparison.

### OAuth via Entra ID App Registration with OBO ✅ Recommended

- The connector authenticates on behalf of the signed-in user using the OAuth 2.0 On-Behalf-Of (OBO) grant against Entra ID.
- Required when using Copilot Studio with MCP Tools and a custom connector, so that Databricks can enforce per-user access controls.
- Tokens are short-lived and automatically refreshed; supports Entra ID conditional access and audit logging.
- See [pattern-oauth-obo](../../20-patterns/authentication/pattern-oauth-obo.md).

### OAuth via Entra ID App Registration (Client Credentials)

- The connector authenticates as a service principal (application identity) using the OAuth 2.0 client credentials grant against Entra ID.
- Suitable for background flows and Copilot Studio actions that do not require an interactive user.
- No user identity is passed to Databricks; Databricks sees the service principal, not the end user.
- See [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md).

### OAuth via Databricks App Connection

- Databricks-native OAuth application connection, issued by the Databricks workspace rather than Entra ID.
- May be appropriate when the workspace is not integrated with Entra ID.
- Does not benefit from Entra ID conditional access or centralized identity governance.
- **Not tested** in the configurations documented here.

### API Key / PAT (Personal Access Token)

- Databricks Personal Access Token passed as a Bearer token.
- Simple to configure but scoped to a single user; not recommended for production.
- **Not tested** in the configurations documented here; cannot be vouched for. Use OAuth 2.0 for validated deployments.

## Configuration Steps

### Part 1: Entra App Registration

Perform these steps in the [Azure portal](https://portal.azure.com) before configuring any Power Platform connector or MCP Tool.

1. **Create a single-tenant app registration.**
   - In the Azure portal, navigate to **Microsoft Entra ID → App registrations → New registration**.
   - Enter a display name (e.g., `CopilotStudio-Databricks-Connector`).
   - Set **Supported account types** to *Accounts in this organizational directory only (single tenant)*.
   - Leave the redirect URI blank for now (it will be added in Part 2).
   - Click **Register** and note the **Application (client) ID** and **Directory (tenant) ID**.

2. **Add the AzureDatabricks delegated API permission and grant admin consent.**
   - In the app registration, navigate to **API permissions → Add a permission**.
   - Choose **APIs my organization uses** and search for `AzureDatabricks`.
   - Select **Delegated permissions** and check `user_impersonation`.
   - Click **Add permissions**.
   - Click **Grant admin consent for \<your tenant\>** and confirm. The status column should show a green check mark.

3. **Expose an API (required for the OBO flow).**
   - Navigate to **Expose an API**.
   - Click **Add** next to the Application ID URI field. Accept the default `api://<client-id>` value or provide a custom URI, then click **Save**.
   - Click **Add a scope** and fill in the following fields:
     | Field | Example value |
     |---|---|
     | Scope name | `access_as_user` |
     | Who can consent | **Admins and users** |
     | Admin consent display name | `Access Databricks as the signed-in user` |
     | Admin consent description | `Allows the application to call Databricks on behalf of the signed-in user.` |
     | User consent display name | `Access Databricks as you` |
     | User consent description | `Allows the application to call Databricks on your behalf.` |
     | State | **Enabled** |
   - Click **Add scope**.

4. **Authorize the Azure API Connections service principal.**
   - Still on the **Expose an API** page, scroll to **Authorized client applications** and click **Add a client application**.
   - Enter the client ID `fe053c5f-3692-4f14-aef2-ee34fc081cae`. This is the well-known Application ID for the **Azure API Connections** service, which Power Platform uses when it exchanges tokens during the OBO flow.
     See [Configure Power Apps authentication settings for your custom connector](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-custom-connector-on-behalf-of#configure-power-apps-authentication-settings-for-your-custom-connector) for details.
   - Check the `api://<client-id>/access_as_user` scope created in the previous step.
   - Click **Add application**.

5. **Create a client secret.**
   - Navigate to **Certificates & secrets → New client secret**.
   - Enter a description and choose an expiry period.
   - Click **Add** and **immediately copy the secret value** — it will not be shown again.
   - Make note of the following three values; you will need them when configuring the connector:
     - **Tenant ID** (Directory ID)
     - **Client ID** (Application ID)
     - **Client secret** (the value copied above)

### Part 2: Post-MCP-Tool-Setup Steps

After completing Part 1, create or import your MCP Tool in Copilot Studio. During that process, a **redirect URI** will be presented by the tool creation wizard. Return to the app registration to add it:

1. **Add the redirect URI.**
   - In the app registration, navigate to **Authentication → Add a platform → Web**.
   - Paste the redirect URI provided by the MCP Tool creation wizard.
   - Click **Configure**.

2. **Configure OBO and Resource URI on the custom connector.**
   The MCP Tool creation process generates a custom connector. This connector must be updated to enable OBO and supply the correct Resource URI.
   See [Configure Power Apps authentication settings for your custom connector](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-custom-connector-on-behalf-of#configure-power-apps-authentication-settings-for-your-custom-connector) for the full Microsoft documentation.

   - From the maker portal ([make.powerapps.com](https://make.powerapps.com)), navigate to **Custom connectors** in the same environment as your Copilot Studio agent.
   - Locate the connector created by the MCP Tool process and click **Edit (pencil icon)**.
   - Go to the **Security** tab.
   - Set **Authentication type** to **OAuth 2.0** if not already selected.
   - Under **OAuth 2.0**, fill in the following fields:
     | Field | Value |
     |---|---|
     | Identity Provider | Azure Active Directory |
     | Client id | Application (client) ID from Part 1 |
     | Client secret | Secret value from Part 1 |
     | Tenant ID | Directory (tenant) ID from Part 1 |
     | Resource URL | `api://<client-id>` (the Application ID URI set in step 3 of Part 1) |
     | Enable on-behalf-of login | **Enabled** |
   - Click **Update connector**.

## Validation Steps

1. Open the connector connection in the Power Platform maker portal.
2. Click **Test connection** — expect no error and a successful sign-in prompt if OBO is enabled.
3. In a test flow, call the connector and inspect the response for HTTP 200.
4. Verify the token audience (`aud`) claim in the JWT matches `api://<client-id>` (the Application ID URI).
5. Verify the `oid` or `upn` claim in the JWT corresponds to the expected user when using OBO.

## Known Limitations

- OOB connectors do not support OBO; a **custom connector** is required for the OBO flow.
- Client secret rotation must be coordinated between Entra ID and the Power Platform connection.
- The redirect URI must be registered in the app registration before users can complete the sign-in flow.

## Related

- [component-oob-connector-behavior](../connectors/component-oob-connector-behavior.md)
- [component-custom-connector-auth](../connectors/component-custom-connector-auth.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
- [pattern-oauth-obo](../../20-patterns/authentication/pattern-oauth-obo.md)
- [Microsoft Docs: Configure a custom connector for OBO](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-custom-connector-on-behalf-of)
