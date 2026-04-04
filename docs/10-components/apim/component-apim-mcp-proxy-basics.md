# Component: APIM MCP Proxy Basics

## Purpose

Describes how to configure Azure API Management (APIM) as a transparent proxy for Databricks MCP endpoints (`mcpsql`, `mcpgenie`), including API definition, routing, and authentication pass-through.

## When to Use

Reference this component when:
- You want APIM to front one or more Databricks MCP endpoints.
- You need to expose MCP endpoints to Power Platform with a stable, APIM-managed URL.
- You require APIM-level policies (throttling, transformation, logging) on MCP traffic.

## Inputs / Prerequisites

- APIM instance provisioned (any SKU for basic proxy; Developer, Standard, or Premium for VNet integration).
- Databricks workspace URL and MCP endpoint paths known. TODO: confirm exact paths for `mcpsql` and `mcpgenie`.
- Backend authentication credential (OAuth token or PAT) available for APIM to use when calling Databricks.
- OpenAPI spec for the Databricks MCP endpoint, or manually defined operations.

## Outputs / What "Done" Looks Like

- APIM API is created with operations matching the Databricks MCP endpoint.
- Test call from APIM test console returns a valid MCP response.
- Power Platform connector can invoke APIM URL and receive expected MCP response.

## Configuration Steps

1. In the Azure portal, navigate to your APIM instance.
2. Create a new API:
   - Select **Add API > HTTP** (or import OpenAPI spec if available).
   - Set **Web service URL** to the Databricks workspace endpoint: `https://<workspace>.azuredatabricks.net/`.
   - Set **API URL suffix** to a meaningful path, e.g., `mcp`.
3. Add operations for each MCP action (e.g., `POST /mcp/call`, `GET /mcp/tools`). TODO: confirm exact MCP endpoint operations.
4. Configure an inbound policy to inject the Databricks auth token:
   ```xml
   <set-header name="Authorization" exists-action="override">
     <value>Bearer {{databricks-token}}</value>
   </set-header>
   ```
   Store the token in an APIM Named Value (`databricks-token`).
5. Configure CORS policy if Power Platform requires it. TODO: confirm CORS requirements.
6. Test from the APIM test console.

## Validation Steps

1. In the APIM test console, invoke a test operation — expect HTTP 200 and a valid MCP response body.
2. Check APIM trace to confirm the `Authorization` header is being forwarded to Databricks.
3. Invoke from the Power Platform connector — expect HTTP 200.
4. Rotate the `databricks-token` Named Value and re-test to confirm the new token is picked up.

## Known Limitations

- APIM Named Values do not auto-rotate; token expiry must be managed externally (e.g., via Azure Key Vault reference with periodic refresh). TODO: confirm Key Vault integration steps.
- SSE (Server-Sent Events) and WebSocket streaming from Databricks MCP are not transparently proxied by APIM by default. TODO: confirm streaming support.
- APIM adds latency (~10–50 ms typical); profile under load if latency is critical.

## Related

- [component-apim-private-ingress-egress](component-apim-private-ingress-egress.md)
- [component-dbx-mcpsql](../databricks/component-dbx-mcpsql.md)
- [component-dbx-mcpgenie](../databricks/component-dbx-mcpgenie.md)
- [pattern-private-via-apim](../../20-patterns/connectivity/pattern-private-via-apim.md)
