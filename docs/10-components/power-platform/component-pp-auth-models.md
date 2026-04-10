# Component: Power Platform Authentication Models

## Purpose

Describes the authentication models supported by Power Platform connectors when calling external APIs (including Databricks MCP endpoints), and how to configure them.

## When to Use

Reference this component when:
- Choosing an authentication scheme for a new connector connection.
- Troubleshooting 401/403 errors from Power Platform to Databricks.
- Understanding the trade-offs between application identity (OAuth) and API key authentication.

## Inputs / Prerequisites

- Entra ID (Azure AD) tenant with permission to register app registrations.
- Databricks workspace configured to accept OAuth tokens from the Entra ID tenant.
- Client ID, client secret (or certificate), and tenant ID for the app registration.

## Outputs / What "Done" Looks Like

- Connector connection successfully authenticates and receives a valid access token.
- Databricks API calls return HTTP 200 (not 401/403).
- Token expiry and refresh behave correctly without manual intervention.

## Supported Auth Models

Three authentication options are available when connecting to Databricks. **OAuth via an Entra ID app registration is the recommended approach.** See [component-dbx-auth](../databricks/component-dbx-auth.md) for a full comparison.

### OAuth via Entra ID App Registration ✅ Recommended

- The connector authenticates as a service principal (application identity) using the OAuth 2.0 client credentials grant against Entra ID.
- Suitable for background flows and Copilot Studio actions that do not require an interactive user.
- Tokens are short-lived and automatically refreshed; supports Entra ID conditional access and audit logging.
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

1. Register an app in Entra ID with the required API permissions for Databricks. TODO: confirm required permissions.
2. Create a client secret (or upload a certificate).
3. Grant the app registration access in the Databricks workspace (e.g., as a service principal in the workspace).
4. In the Power Platform connector connection, select the OAuth scheme and supply the client ID, secret, and tenant ID.
5. Test the connection (see Validation below).

## Validation Steps

1. Open the connector connection in the Power Platform maker portal.
2. Click "Test connection" — expect no error.
3. In a test flow, call the connector and inspect the response for HTTP 200.
4. Verify the token audience (`aud`) matches the Databricks resource. TODO: confirm expected `aud` value.

## Known Limitations

- OOB connectors may not support all OAuth grant types; check the connector documentation.
- Client secret rotation must be coordinated between Entra ID and the Power Platform connection.

## Related

- [component-oob-connector-behavior](../connectors/component-oob-connector-behavior.md)
- [component-custom-connector-auth](../connectors/component-custom-connector-auth.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
- [pattern-oauth-obo](../../20-patterns/authentication/pattern-oauth-obo.md)
