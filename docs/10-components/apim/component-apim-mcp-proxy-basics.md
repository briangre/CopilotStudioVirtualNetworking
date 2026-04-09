# Component: APIM MCP Proxy Basics

## Purpose

Describes how to configure Azure API Management (APIM) to expose Databricks MCP endpoints (`mcpsql`, `mcpgenie`) to Power Platform with a stable, APIM-managed URL.

## When to Use

Reference this component when:
- You want APIM to front one or more Databricks MCP endpoints.
- You need to expose MCP endpoints to Power Platform with a stable, APIM-managed URL.
- You require APIM-level policies (throttling, transformation, logging) on MCP traffic.

## Recommended Approach: Use the APIM "Expose Existing MCP Server" Feature

> **Note:** Azure API Management now includes a **preview feature** that lets you expose an existing MCP server directly — without manually building each API operation. This is the recommended approach for this solution.

📖 **Follow the official Microsoft Learn guide:**
**[Expose an existing MCP server using Azure API Management (Preview)](https://learn.microsoft.com/en-us/azure/api-management/expose-existing-mcp-server)**

### Why this is better than a manual API build

- **No manual operation definition** — APIM discovers and proxies MCP tools automatically.
- **Maintained by Microsoft** — the integration stays aligned with MCP spec changes and APIM updates.
- **Simpler policy surface** — authentication pass-through, CORS, and throttling policies can still be applied at the APIM gateway level.
- **Faster to set up** — point APIM at your Databricks MCP endpoint URL and the feature handles the rest.

### High-level steps

1. Provision an APIM instance. Any SKU works for basic proxying; Developer, Standard, or Premium is recommended for VNet integration. Check the [APIM SKU feature comparison](https://learn.microsoft.com/en-us/azure/api-management/api-management-features) for any preview-specific SKU requirements.
2. Follow the [Microsoft Learn guide](https://learn.microsoft.com/en-us/azure/api-management/expose-existing-mcp-server) to import your Databricks MCP endpoint (`mcpsql` or `mcpgenie`) as an MCP-backed API. Authentication to Databricks is handled automatically.
3. Wire up the API to your Power Platform custom connector. **Note:** Because the "Expose existing MCP server" feature is currently in preview, there is no built-in APIM test pane for this API type. Valid options for testing are:
   - Deploy a VM inside the inbound APIM VNet and test the endpoint directly from that VM.
   - Test end-to-end by invoking the custom connector from Power Platform.

## Related

- [component-apim-private-ingress-egress](component-apim-private-ingress-egress.md)
- [component-dbx-mcpsql](../databricks/component-dbx-mcpsql.md)
- [component-dbx-mcpgenie](../databricks/component-dbx-mcpgenie.md)
- [pattern-private-via-apim](../../20-patterns/connectivity/pattern-private-via-apim.md)
