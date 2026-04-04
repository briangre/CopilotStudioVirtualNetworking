# Component: Power Platform Authentication Models

## Purpose

Describes the authentication models supported by Power Platform connectors when calling external APIs (including Databricks MCP endpoints), and how to configure them.

## When to Use

Reference this component when:
- Choosing an authentication scheme for a new connector connection.
- Troubleshooting 401/403 errors from Power Platform to Databricks.
- Understanding how delegated vs. application identity flows work.

## Inputs / Prerequisites

- Entra ID (Azure AD) tenant with permission to register app registrations.
- Databricks workspace configured to accept OAuth tokens from the Entra ID tenant.
- Client ID, client secret (or certificate), and tenant ID for the app registration.

## Outputs / What "Done" Looks Like

- Connector connection successfully authenticates and receives a valid access token.
- Databricks API calls return HTTP 200 (not 401/403).
- Token expiry and refresh behave correctly without manual intervention.

## Supported Auth Models

### OAuth 2.0 – Client Credentials (Application Identity)

- The connector authenticates as an application (service principal), not as a user.
- Suitable for background flows and Copilot Studio actions that do not have an interactive user.
- Token scope: `TODO: confirm Databricks OAuth scope`.
- See [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md).

### OAuth 2.0 – On-Behalf-Of (OBO) / Delegated

- The connector passes the signed-in user's identity through to Databricks.
- Requires the calling user to have been granted access in the Databricks workspace.
- Suitable for Power Apps where the end-user identity matters for data access control.
- See [pattern-oauth-obo](../../20-patterns/authentication/pattern-oauth-obo.md).

### API Key / PAT (Personal Access Token)

- Databricks Personal Access Token passed as a Bearer token.
- Simple to configure but scoped to a single user; not recommended for production.
- TODO: confirm whether OOB connector supports PAT.

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
- OBO flow requires the user to have previously consented to the app registration.

## Related

- [component-oob-connector-behavior](../connectors/component-oob-connector-behavior.md)
- [component-custom-connector-auth](../connectors/component-custom-connector-auth.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
- [pattern-oauth-obo](../../20-patterns/authentication/pattern-oauth-obo.md)
