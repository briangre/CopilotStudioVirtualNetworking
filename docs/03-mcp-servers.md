# MCP Server Architecture and Copilot Studio Integration

## What MCP Is

Model Context Protocol (MCP) is an open protocol that standardizes how AI models communicate with external tools and data sources. Copilot Studio supports MCP as a native agent action type.

MCP defines:
- A **manifest** that describes available tools and their input/output schemas
- A **tool call** request/response pattern (HTTP POST with structured JSON)
- Optional **streaming** responses (SSE)

---

## What MCP Is Not

MCP is not a REST API wrapper. It is not a Databricks feature. Databricks does not ship an MCP server.

This means: to use MCP with Databricks, you must build or operate a service that:
1. Implements the MCP protocol
2. Calls Databricks APIs internally on behalf of the agent
3. Is reachable via HTTPS from Power Platform

This is an **architectural requirement**, not a configuration option.

---

## Supported MCP Configuration in Copilot Studio

**Status: ✅ Supported for public HTTPS endpoints**

Copilot Studio supports adding MCP servers as agent actions via:
- A manifest URL (e.g., `https://your-mcp-server.com/.well-known/mcp.json`)
- Tool discovery from the manifest
- Individual tool invocations at runtime

### Requirements

| Requirement | Detail |
|---|---|
| Protocol | HTTPS (TLS 1.2+) |
| Authentication | Bearer token (Entra ID) or API key passed in headers |
| Manifest format | Must conform to MCP spec version supported by Copilot Studio |
| Network accessibility | Must be reachable from Power Platform's outbound IP ranges |
| Private endpoint | ❌ Not supported without APIM in front |

---

## MCP Server Architecture Options

### Option 1: Publicly Hosted MCP Server

**Status: ✅ Recommended for most scenarios**

```
Copilot Studio
    │
    │  HTTPS (Bearer token)
    ▼
MCP Server (Azure App Service / Container App / AKS)
    │
    │  REST API (Bearer token)
    ▼
Databricks Workspace (public or private endpoint)
```

The MCP server acts as the translator between the MCP protocol and Databricks REST APIs.

**What the MCP server handles:**
- Receiving MCP tool calls from Copilot Studio
- Acquiring or validating the Databricks API token
- Calling the appropriate Databricks API (SQL Statement, Genie, Jobs, etc.)
- Formatting the response as an MCP tool response

**Hosting options:**
- Azure App Service (simplest, supports HTTPS out-of-box)
- Azure Container Apps (good for containerized MCP servers)
- Azure Functions (for lightweight, low-frequency scenarios)
- Azure Kubernetes Service (for high-scale or multi-tenant deployments)

---

### Option 2: MCP Server with APIM Proxy

**Status: ⚠️ Supported with additional complexity**

Use this when the MCP server itself is in a private VNet, or when you need:
- Centralized auth policy (token validation at APIM)
- Rate limiting or quota management
- Logging and observability at the API gateway layer

```
Copilot Studio
    │
    │  HTTPS (Bearer token)
    ▼
Azure API Management (public endpoint)
    │
    │  Internal VNet routing
    ▼
MCP Server (private, VNet-integrated)
    │
    │  Private Link / VNet peering
    ▼
Databricks Workspace (private endpoint)
```

See [05-apim.md](./05-apim.md) for the full APIM configuration guide.

---

### Option 3: Direct Databricks API as "MCP" — Not Viable

**Status: ❌ Not supported**

Some architects attempt to configure the Databricks REST API as an MCP server by pointing the manifest URL at Databricks directly. This does not work because:

- Databricks does not serve an MCP manifest
- Databricks API response formats are not MCP tool responses
- There is no SSE streaming at the Databricks API layer matching MCP expectations

**This is a protocol mismatch, not a configuration issue.**

---

## Building an MCP Server for Databricks

### Minimum Viable MCP Server Structure

An MCP server for Databricks needs to expose at minimum:

```json
// /.well-known/mcp.json
{
  "schema_version": "v1",
  "name": "databricks-mcp",
  "description": "MCP server for Azure Databricks",
  "tools": [
    {
      "name": "run_sql_query",
      "description": "Execute a SQL query against a Databricks SQL Warehouse",
      "input_schema": {
        "type": "object",
        "properties": {
          "query": { "type": "string" },
          "warehouse_id": { "type": "string" }
        },
        "required": ["query", "warehouse_id"]
      }
    }
  ]
}
```

### Tool Call Handler (Python Example Pattern)

```python
@app.post("/tools/run_sql_query")
async def run_sql_query(request: ToolCallRequest, token: str = Depends(verify_token)):
    # 1. Validate input
    query = request.input["query"]
    warehouse_id = request.input["warehouse_id"]

    # Security: Do NOT interpolate query strings directly into SQL.
    # Always pass queries to Databricks using the parameter binding mechanism
    # (`:param` syntax in the SQL Statement API) or enforce a pre-approved
    # query allowlist here before calling execute_statement.
    # Passing raw, user-supplied SQL to Databricks creates a SQL injection risk.

    # 2. Call Databricks SQL Statement API
    result = await databricks_client.execute_statement(
        statement=query,
        warehouse_id=warehouse_id
    )

    # 3. Return MCP-formatted response
    return {
        "output": {
            "columns": result.schema.column_names,
            "rows": result.result.data_array
        }
    }
```

### Security Considerations

- The MCP server must validate the caller's token before processing any tool call
- Never pass raw Databricks credentials in the MCP manifest URL or tool parameters
- Use managed identity or a dedicated service principal for the MCP server → Databricks connection
- Do not expose warehouse IDs or catalog names in the MCP manifest if they are sensitive

---

## Authentication Between Copilot Studio and the MCP Server

Copilot Studio passes authentication to MCP servers via HTTP headers. Supported patterns:

| Pattern | How |
|---|---|
| API Key | Configure in Copilot Studio MCP connection; passed as header |
| OAuth 2.0 Bearer | Entra ID app; Copilot Studio acquires token and sends in `Authorization` header |
| No auth (dev/test only) | ❌ Do not use in production |

For production deployments, use Entra ID OAuth. See [06-auth.md](./06-auth.md).

---

## Common Failure Modes

### "MCP manifest not found"
- Verify the manifest URL is reachable from Power Platform's outbound IPs
- Check CORS headers if your server returns them
- Confirm the manifest content-type is `application/json`

### "Tool call returns 401"
- The token passed by Copilot Studio is not being validated or is misconfigured
- Check the Entra ID app registration audience matches the MCP server's expected audience

### "Tool call succeeds but agent ignores result"
- The tool response format may not match what Copilot Studio expects
- Ensure the response includes an `output` field with the data
- Large result sets may be truncated — consider summarizing in the MCP server layer

### "MCP server is unreachable (private endpoint)"
- Copilot Studio cannot reach private endpoints without an APIM proxy
- This is a product limitation, not a networking misconfiguration you can resolve on the Copilot Studio side
- See [05-apim.md](./05-apim.md)
