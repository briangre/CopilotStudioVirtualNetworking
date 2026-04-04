# Component: Databricks Authentication

## Purpose

Describes the authentication options for calling Databricks MCP endpoints, including OAuth 2.0 (service principal) and Personal Access Tokens (PAT), and how to configure each.

## When to Use

Reference this component when:
- Choosing the auth method for a connector connection to Databricks.
- Configuring a service principal for application-identity access to Databricks.
- Rotating or managing credentials for Databricks access.

## Inputs / Prerequisites

- Databricks workspace with Unity Catalog or legacy workspace access configured.
- Entra ID tenant (for OAuth 2.0 service principal flow).
- App registration created in Entra ID. See [component-pp-auth-models](../power-platform/component-pp-auth-models.md).
- Service principal added to the Databricks workspace with appropriate permissions.

## Outputs / What "Done" Looks Like

- A valid OAuth 2.0 Bearer token (or PAT) can be used to call the Databricks MCP endpoint.
- Token (or PAT) is accepted by the Databricks API (HTTP 200 on an authenticated endpoint).
- For OAuth: token refresh works without manual intervention.

## Authentication Options

### Option 1: OAuth 2.0 – Client Credentials (Recommended for Production)

1. Register a service principal in Entra ID.
2. In the Databricks workspace, add the service principal:
   - Navigate to **Settings > Identity and access > Service principals > Add service principal**.
   - Grant the service principal appropriate roles (e.g., "Can use" on SQL Warehouse, data access on catalogs).
3. Configure the token endpoint: `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token`
4. Token scope: TODO: confirm Databricks OAuth scope (likely `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default` for Azure Databricks).
5. Use the obtained token as `Authorization: Bearer <token>` in API calls.

### Option 2: Databricks Personal Access Token (PAT) — Development Only

1. In the Databricks workspace, navigate to **Settings > Developer > Access tokens > Generate new token**.
2. Set an expiry and note the token value (shown only once).
3. Use the PAT as `Authorization: Bearer <pat>` in API calls.
4. **Not recommended for production**; PATs are user-scoped and long-lived.

## Validation Steps

1. Using curl, request a token from the Entra ID token endpoint (for OAuth option):
   ```bash
   curl -X POST https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token \
     -d "grant_type=client_credentials&client_id={client-id}&client_secret={secret}&scope=2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default"
   ```
   Expect a JSON response with `access_token`.
2. Use the token to call a Databricks API:
   ```bash
   curl -H "Authorization: Bearer <token>" \
     https://<workspace>.azuredatabricks.net/api/2.0/clusters/list
   ```
   Expect HTTP 200 or 403 (not 401 — 401 indicates the token is invalid).
3. Confirm the service principal appears in the Databricks audit log for the API call.

## Known Limitations

- OAuth tokens expire; the connector or calling code must handle token refresh.
- PATs do not expire by default if no expiry is set; set an explicit expiry.
- Service principal permissions must be explicitly granted; they do not inherit group permissions automatically in all scenarios. TODO: confirm group behavior.
- Databricks OAuth scope value may differ by region or deployment type. TODO: confirm.

## Related

- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [component-dbx-mcpsql](component-dbx-mcpsql.md)
- [component-dbx-mcpgenie](component-dbx-mcpgenie.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
