# Component: Databricks MCP Genie Endpoint (mcpgenie)

## Purpose

Describes the Databricks MCP endpoint backed by **AI/BI Genie** (`mcpgenie`), its natural-language query capabilities, and how to configure it for use by Power Platform connectors.

## When to Use

Reference this component when:
- You need a Power Platform agent to answer natural-language questions about data using Databricks AI/BI Genie.
- You are configuring the `mcpgenie` endpoint URL and authentication for a connector.

## Inputs / Prerequisites

- Databricks workspace with AI/BI Genie enabled and a Genie Space configured.
- Genie Space ID noted.
- SQL Warehouse associated with the Genie Space is running.
- Service principal (or PAT) with access to the Genie Space.
- MCP server feature enabled for the Genie Space. TODO: confirm how to enable MCP server for Genie.
- Workspace URL: `https://<workspace-id>.azuredatabricks.net`.

## Outputs / What "Done" Looks Like

- MCP `tools/list` call returns the Genie-specific tools (e.g., `ask_question`).
- MCP `tools/call` with a natural-language question returns a meaningful answer.
- Authentication token is accepted (no 401/403).

## Endpoint Details

| Property | Value |
|----------|-------|
| Endpoint path | TODO: confirm exact MCP Genie path |
| Protocol | HTTP/HTTPS + Server-Sent Events (SSE) or HTTP streaming |
| Auth | OAuth 2.0 Bearer token or Databricks PAT |
| Genie Space | Must exist and be accessible to the service principal |

## Configuration Steps

1. In the Databricks workspace, navigate to **AI/BI > Genie** and create or identify a Genie Space.
2. Note the **Genie Space ID**.
3. Enable the MCP server for the Genie Space. TODO: confirm steps (UI or API).
4. Note the resulting MCP endpoint URL. TODO: confirm URL format, expected to include the Space ID.
5. Grant the service principal access to the Genie Space.
6. Configure the Power Platform connector with the MCP endpoint URL and auth. See [component-dbx-auth](component-dbx-auth.md).

## Validation Steps

1. Using a REST client, call the MCP `tools/list` endpoint:
   ```
   GET <mcp-endpoint>/tools/list
   Authorization: Bearer <token>
   ```
   Expect HTTP 200 and a JSON array including a natural-language query tool.
2. Call `tools/call` with a natural-language question (e.g., "What are the top 10 customers by revenue?") and verify a meaningful response.
3. Confirm the Genie Space query history shows the question was processed.

## Known Limitations

- AI/BI Genie requires a Databricks workspace with Unity Catalog enabled. TODO: confirm.
- Response quality depends on the data assets and instructions configured in the Genie Space.
- Genie is not a SQL pass-through; complex or unsupported queries may return an error or a partial answer.
- PAT tokens are user-scoped and not recommended for production automation.

## Related

- [component-dbx-mcpsql](component-dbx-mcpsql.md)
- [component-dbx-auth](component-dbx-auth.md)
- [pattern-dbx-mcp-genie](../../20-patterns/databricks/pattern-dbx-mcp-genie.md)
