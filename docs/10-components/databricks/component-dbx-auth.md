# Component: Databricks Authentication

## Purpose

Provides a high-level overview of the authentication options available when connecting a Copilot Studio agent to an Azure Databricks workspace — either via an MCP tool or the out-of-the-box (OOB) Azure Databricks connector — targeting the Genie or SQL MCP endpoints.

This document covers the known options and identifies the recommended approach. It does not replicate Databricks or Entra ID documentation; refer to the official product docs for step-by-step configuration details.

## When to Use

Reference this component when:
- Choosing an authentication scheme for a connector or MCP tool connection to Databricks.
- Evaluating the trade-offs between OAuth and API key authentication.

## Authentication Options

Three authentication options are available for this scenario. **OAuth using an Entra ID app registration is the recommended approach.**

### Option 1: OAuth via Entra ID App Registration ✅ Recommended

An app registration (service principal) in Entra ID is granted access to the Databricks workspace. The connector or MCP tool uses the OAuth 2.0 client credentials grant to obtain a token from Entra ID and presents it as a Bearer token to the Databricks endpoint.

- **Identity**: Application identity (service principal); no end-user delegation.
- **Token issuer**: Entra ID (`login.microsoftonline.com`).
- **Why recommended**: Tokens are short-lived, support automatic refresh, and follow enterprise identity governance with Entra ID. Auditing and conditional access policies can be applied centrally.
- **Tested**: Yes — this option has been validated in the configurations documented here.

See [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md) for the full authentication flow.

### Option 2: OAuth via Databricks App Connection

Databricks supports its own OAuth application connections, which can be used instead of Entra ID as the identity provider. This may be appropriate when the Databricks workspace is not integrated with Entra ID or when workspace-native identity management is preferred.

- **Identity**: Databricks-managed application identity.
- **Token issuer**: Databricks workspace (not Entra ID).
- **Trade-off**: Databricks-native OAuth does not benefit from Entra ID conditional access or centralized identity governance.
- **Tested**: Not validated in the configurations documented here.

### Option 3: API Key (Personal Access Token)

A Databricks Personal Access Token (PAT) is passed as a Bearer token directly. This is the simplest option to configure but carries significant limitations.

- **Identity**: Scoped to the individual Databricks user who generated the token.
- **Not recommended for production**: PATs are long-lived (if no expiry is set), user-scoped, and not tied to enterprise identity controls.
- **Tested**: Not validated in the configurations documented here.

## Summary Comparison

| Option | Identity type | Token issuer | Recommended? | Tested? |
|--------|---------------|--------------|--------------|---------|
| OAuth via Entra ID | Service principal | Entra ID | ✅ Yes | ✅ Yes |
| OAuth via Databricks App Connection | App connection | Databricks | No | No |
| API Key (PAT) | User-scoped | N/A | No | No |

## Known Limitations

- OAuth tokens expire; the connector or MCP tool must handle token refresh.
- Service principal permissions in the Databricks workspace must be explicitly granted.
- PATs are user-scoped and long-lived if no expiry is set; avoid for production deployments.

## Related

- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [component-dbx-mcpsql](component-dbx-mcpsql.md)
- [component-dbx-mcpgenie](component-dbx-mcpgenie.md)
- [pattern-oauth-client-credentials](../../20-patterns/authentication/pattern-oauth-client-credentials.md)
