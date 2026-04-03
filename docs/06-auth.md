# Entra ID App Registration and OAuth Flows

## Why This Is Complicated

Copilot Studio, Power Platform connectors, and Databricks all interact with Entra ID (formerly Azure AD) — but they do so in different ways, at different times, and with different token audiences. Misalignment between these is the most common cause of `401 Unauthorized` errors in this integration.

This guide maps the auth flow end-to-end so you can identify exactly where a failure is occurring.

---

## Auth Principals Involved

| Principal | What It Is | Used For |
|---|---|---|
| **User identity** | The person using the Copilot Studio agent | Delegated auth flows (on-behalf-of) |
| **Copilot Studio service** | Microsoft-managed; not configurable by you | Making API calls from agent actions |
| **Power Platform connector identity** | The connection's configured credential | Authenticating custom connector calls |
| **APIM Managed Identity** | System-assigned identity on the APIM instance | APIM acquiring Databricks tokens |
| **MCP server identity** | Managed identity or service principal on the MCP host | MCP server calling Databricks APIs |
| **Databricks service principal** | An Entra ID app with Databricks permissions | Non-interactive / service-to-service calls |

---

## Token Flow: Custom Connector → Databricks (Direct)

**Status: ✅ Supported**

```
User / Agent trigger
    │
    ▼
Power Platform Custom Connector
    │  Connection configured with OAuth 2.0 (Entra ID)
    │  Token audience: 2ff814a6-3304-4ab8-85cb-cd0e6f879c1d (Databricks)
    │  Token cached per connection; refreshed automatically
    ▼
Databricks REST API
    │  Validates token against Azure Databricks audience
    ▼
Result
```

### Configuring the Custom Connector OAuth

In the custom connector authentication settings:

| Field | Value |
|---|---|
| Identity provider | Azure Active Directory |
| Client ID | Your app registration client ID |
| Client secret | Your app registration secret (or cert) |
| Authorization URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize` |
| Token URL | `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token` |
| Scope | `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default` |
| Redirect URL | Power Platform's OAuth redirect (auto-populated) |

> ⚠️ The scope `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default` is the Azure Databricks first-party app ID. This is correct for service-to-service (client credentials) flows. Do not use `user_impersonation` unless you specifically need delegated access.

---

## Token Flow: Copilot Studio → APIM → Databricks

**Status: ✅ Supported (recommended for private workspaces)**

This is a two-leg auth flow:

```
Copilot Studio Action
    │  Token audience: api://{apim-app-registration-id}
    ▼
APIM (validates token, then acquires Databricks token)
    │  [Managed Identity] → Databricks token audience
    │  OR
    │  [Named Value PAT] → injected as Bearer header
    ▼
Databricks Private Endpoint
```

### Leg 1: Copilot Studio → APIM

Create an **Entra ID app registration** for APIM:
- App ID URI: `api://{your-apim-client-id}` (or a custom URI)
- Expose an API with a scope (e.g., `Databricks.Query`)
- Grant this scope to the Power Platform connector or MCP server identity

Copilot Studio / the connector acquires a token for this app and sends it to APIM.

APIM validates the token using the `validate-jwt` policy (see [05-apim.md](./05-apim.md)).

### Leg 2: APIM → Databricks

Option A — Managed Identity (recommended):
- Assign a system-managed identity to the APIM instance
- In Databricks, add the managed identity as a workspace user or group member with appropriate permissions
- In the APIM policy, use `<authentication-managed-identity resource="2ff814a6-3304-4ab8-85cb-cd0e6f879c1d" />`

Option B — Service Principal with Client Credentials:
- Create a Databricks service principal in Entra ID
- Store the client secret in Azure Key Vault
- In APIM, use a `send-request` policy to acquire a token at request time
- Inject the token as the `Authorization` header toward Databricks

Option C — Personal Access Token (simplest but not recommended for production):
- Generate a PAT in Databricks
- Store in APIM Named Values (ideally referencing Key Vault)
- Inject via `set-header` policy

---

## Token Flow: Copilot Studio → MCP Server → Databricks

**Status: ✅ Supported**

```
Copilot Studio
    │  Token audience: api://{mcp-server-app-id}
    ▼
MCP Server
    │  [Validates inbound token]
    │  [Acquires Databricks token via managed identity or service principal]
    ▼
Databricks
```

### App Registration for the MCP Server

1. Register an app in Entra ID for the MCP server
2. Expose an API scope (e.g., `MCP.Call`)
3. In Copilot Studio's MCP connection configuration, set the OAuth audience to this app's URI
4. The MCP server validates the inbound token's `aud` claim against its app ID

The MCP server then uses its own identity (managed identity if hosted on Azure) to call Databricks.

---

## Entra ID App Registration Checklist

When setting up for any of the above flows, verify:

- [ ] App registration created in the correct Entra ID tenant
- [ ] Redirect URI added for Power Platform OAuth callback (if using delegated flow)
- [ ] API permissions include the Databricks scope (`2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default` for client credentials)
- [ ] Admin consent granted for Databricks permissions (required for client credentials flow)
- [ ] Token lifetime policies reviewed (default is fine for most scenarios)
- [ ] Client secret has a defined expiry and rotation procedure in place
- [ ] The app is not granted excessive permissions beyond what the integration requires

---

## Common Auth Failures and Causes

### `401 Unauthorized` from Databricks

| Possible Cause | Check |
|---|---|
| Wrong token audience | Verify token `aud` claim is `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d` |
| Token expired | Token lifetime or clock skew issue; check token `exp` claim |
| Service principal not added to Databricks workspace | Add SP in Databricks → Admin Console → Service Principals |
| PAT expired or revoked | Regenerate PAT and update Named Value or secret |

### `401 Unauthorized` from APIM

| Possible Cause | Check |
|---|---|
| Wrong token audience for APIM | Verify `aud` matches the APIM app registration URI |
| Issuer mismatch | Verify issuer in `validate-jwt` policy matches the actual token issuer |
| Token not being passed | Confirm Copilot Studio / connector is sending `Authorization: Bearer` header |

### `403 Forbidden` from Databricks

| Possible Cause | Check |
|---|---|
| Service principal lacks Databricks permissions | Assign appropriate Databricks group or permissions to the SP |
| SQL warehouse permissions | Ensure SP has `CAN USE` on the target warehouse |
| Unity Catalog permissions | Grant `SELECT` or `EXECUTE` on the relevant catalog/schema/table |

### Token acquired but wrong scope

Databricks does not use granular scopes within Entra ID beyond the `.default` scope for its first-party app. If you need row-level or table-level access control, implement it in Databricks Unity Catalog permissions, not in OAuth scopes.

---

## Token Acquisition: Delegated vs Client Credentials

| Flow | When to Use | Risk |
|---|---|---|
| **Client credentials** (app-to-app) | Service-to-service; agent runs as a service identity | If the service principal is over-privileged, all queries run with elevated access |
| **On-behalf-of (OBO)** | Agent acts as the logged-in user | Requires the user to have Databricks permissions; more complex token flow |
| **Personal Access Token** | Dev/test only | PATs do not expire by default; significant security risk if leaked |

For most agent scenarios, **client credentials** is the correct choice. The agent represents a service, not a specific user.

If your organization requires user-level audit trails in Databricks (who ran which query), the on-behalf-of flow is required but adds significant complexity. Consult your identity team before pursuing this path.
