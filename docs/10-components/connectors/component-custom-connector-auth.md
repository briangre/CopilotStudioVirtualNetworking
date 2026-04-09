# Component: Custom Connector Authentication

## Purpose

Describes how to build and configure a custom Power Platform connector with an authentication scheme suitable for calling Databricks MCP endpoints, including scenarios where the OOB connector is insufficient.

## When to Use

Reference this component when:
- The OOB connector does not support your required auth model or endpoint.
- You need to route traffic through APIM or a private endpoint.
- You need to customize request headers, URL paths, or payload transformation using APIM.

## Inputs / Prerequisites

- OpenAPI 2.0 (Swagger) or OpenAPI 3.0 spec for the Databricks MCP endpoint
-    for these docs the built-in MCP Tool in Copilot Studio will be used
- Entra ID app registration with appropriate permissions.

## Outputs / What "Done" Looks Like

- Custom connector is created and shared within the Power Platform environment.
- A connection to the custom connector authenticates successfully.
- Connector actions can be invoked from Copilot Studio agents.

## Configuration Steps

1. Obtain or author the OpenAPI spec for the target Databricks MCP endpoint.
2. In the Power Platform maker portal, navigate to **Data > Custom connectors > + New custom connector > Import an OpenAPI file**.
3. Upload the spec.
4. On the **Security** tab, configure OAuth 2.0:
   - Grant type: `client_credentials` (for application identity) or `authorization_code` (for delegated/OBO).
   - Token URL: `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token`
   - Client ID and secret from the Entra ID app registration.
   - Scope: TODO: confirm Databricks OAuth scope.
5. On the **Definition** tab, verify actions and parameters match the Databricks API.
6. Click **Create connector**.
7. Create a connection: click **Test > + New connection**, provide credentials, and test an action.

## Validation Steps

1. In the connector test UI, invoke a lightweight action (e.g., list tools).
2. Verify HTTP 200 and a valid response body.
3. Add the connector to a Copilot Studio agent and run it end-to-end.
4. Confirm the token is being sent with the `Authorization: Bearer` header. TODO: confirm header inspection method.

## Known Limitations

- Custom connectors are environment-scoped; they must be re-created or imported into each environment.
- Secrets (client secret) are stored in the connector connection and are not rotated automatically.
- Custom connector definitions can be exported/imported as solution components for ALM.
- Maximum response payload size is limited by Power Platform connector limits. TODO: confirm size limit.

## Related

- [component-oob-connector-behavior](component-oob-connector-behavior.md)
- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [component-apim-mcp-proxy-basics](../apim/component-apim-mcp-proxy-basics.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
