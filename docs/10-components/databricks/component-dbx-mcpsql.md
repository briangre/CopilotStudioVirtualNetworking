# Component: Databricks MCP SQL Endpoint (mcpsql)

## Purpose

Describes the Databricks MCP endpoint backed by a **SQL Warehouse** (`mcpsql`), its capabilities, and how to configure it for use by Power Platform connectors.

## When to Use

Reference this component when:
- You need a Copilot Studio agent to query structured data in a Databricks SQL Warehouse via the MCP protocol.
- You are configuring the `mcpsql` endpoint URL and authentication for a connector.

## Inputs / Prerequisites

- Databricks workspace with a SQL Warehouse provisioned and running.
- SQL Warehouse HTTP Path noted (e.g., `/sql/1.0/warehouses/<id>`).
- Service principal (or PAT) with "Can use" permission on the SQL Warehouse and at least "Data Reader" on the target catalogs/schemas.
- MCP server feature enabled on the workspace. TODO: confirm how to enable MCP server for SQL Warehouse.
- Workspace URL: `https://<workspace-id>.azuredatabricks.net`.

## Outputs / What "Done" Looks Like

- MCP `tools/list` call returns the list of SQL tools exposed by the warehouse.
- MCP `tools/call` with a SQL query returns the expected result set.
- Authentication token is accepted by the endpoint (no 401/403).

## Endpoint Details

| Property | Value |
|----------|-------|
| Endpoint path | TODO: confirm exact MCP SQL path |
| Protocol | HTTP/HTTPS + Server-Sent Events (SSE) or HTTP streaming |
| Auth | OAuth 2.0 Bearer token or Databricks PAT |
| SQL Warehouse | Must be running (auto-start may apply) |

## Configuration Steps

1. Provision a SQL Warehouse in the Databricks workspace (or use an existing one).
2. Note the **HTTP Path** from the warehouse connection details.
3. Enable the MCP server for the SQL Warehouse. TODO: confirm steps (UI or API).
4. Note the resulting MCP endpoint URL. TODO: confirm URL format.
5. Grant the service principal "Can use" on the warehouse and the required data permissions.
6. Configure the Power Platform connector with the MCP endpoint URL and auth. See [component-dbx-auth](component-dbx-auth.md).

## Validation Steps

1. Using a REST client (curl, Postman), call the MCP `tools/list` endpoint:
   ```
   GET <mcp-endpoint>/tools/list
   Authorization: Bearer <token>
   ```
   Expect HTTP 200 and a JSON array of tools.
2. Call `tools/call` with a simple SQL query and verify the result.
3. Confirm from the Databricks workspace query history that the query executed on the expected warehouse.

## Known Limitations

- SQL Warehouse must be running; cold-start latency applies if auto-start is enabled.
- PAT tokens are user-scoped and not recommended for production automation.
- Maximum query result size is bounded by the warehouse and MCP server configuration. TODO: confirm limits.
- SQL Warehouse costs accrue while the warehouse is active.

## Related

- [component-dbx-mcpgenie](component-dbx-mcpgenie.md)
- [component-dbx-auth](component-dbx-auth.md)
- [pattern-dbx-mcp-sql](../../20-patterns/databricks/pattern-dbx-mcp-sql.md)
