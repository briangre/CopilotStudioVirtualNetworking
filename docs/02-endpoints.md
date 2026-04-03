# Endpoint Selection: Genie vs SQL vs MCP

## The Decision

Databricks exposes three distinct interaction models relevant to Copilot Studio agents. Choosing the wrong one for your use case causes either functional failure or a poor user experience that is difficult to fix after build.

---

## At a Glance

| Endpoint Type | What it Does | Returns | Best For |
|---|---|---|---|
| SQL Statement Execution API | Runs a SQL query against a warehouse | Structured rows/columns | Deterministic queries, dashboards, lookups |
| Genie Conversational API | Accepts natural language, generates and runs SQL internally | Conversational text + optional data | Exploratory natural language analytics |
| MCP Server | Tool/function calling via MCP protocol | Structured tool responses | Agent-native function calling, multi-step reasoning |

---

## SQL Statement Execution API

**Status: ✅ Supported via Custom Connector or HTTP action**

### API Reference

```
POST /api/2.0/sql/statements
Host: <workspace>.azuredatabricks.net
Authorization: Bearer <token>

{
  "statement": "SELECT * FROM catalog.schema.table WHERE region = :region",
  "warehouse_id": "<warehouse-id>",
  "parameters": [{"name": "region", "value": "West", "type": "STRING"}]
}
```

### What to know before using it

- Results are **paginated**. The initial response may return a `statement_id` and a status of `RUNNING`. You must poll `/api/2.0/sql/statements/{id}` until status is `SUCCEEDED`.
- This is an **asynchronous** API. Your Power Automate flow or HTTP action must handle polling logic.
- Parameter binding (`:param`) prevents SQL injection. Always use it.
- Result sets are returned as a JSON array of arrays. You must map column names from the schema in the response.

### Limitations

| Scenario | Supported? |
|---|---|
| Long-running queries (>30s) | ⚠️ Requires polling loop in flow |
| Returning results directly to Copilot Studio | ⚠️ Must be serialized as text or JSON in flow |
| Private workspace (no APIM) | ❌ |
| More than ~50 rows rendered in agent | ⚠️ Agent context limits apply |

---

## Genie Conversational API

**Status: ⚠️ Works with caveats**

Genie is Databricks' natural language to SQL engine. It accepts a natural language question and returns a conversational response (and optionally, the underlying data).

### API Flow

```
1. POST /api/2.0/genie/spaces/{space-id}/start-conversation
   Body: { "content": "What were total sales in Q1?" }
   Response: { "conversation_id": "...", "message_id": "..." }

2. GET /api/2.0/genie/spaces/{space-id}/conversations/{conversation_id}/messages/{message_id}
   Poll until status = COMPLETED
   Response includes: text reply + optional query_result

3. (Optional) GET query result using attachment ID
```

### Why this is complex for Copilot Studio

Genie is designed for **human-in-the-loop exploratory analytics**, not for deterministic agent tool calling. Specific issues:

- The response is conversational text, not a structured schema. Parsing varies with query type.
- Multi-turn conversation context is stored server-side in Genie — Copilot Studio would need to manage `conversation_id` state across turns.
- Genie can return ambiguity prompts ("Did you mean X or Y?") — your agent must decide how to handle these.
- Genie spaces must be pre-configured in Databricks. Each space is scoped to specific tables/semantics.

### When to use Genie

Use Genie when:
- Your users will ask **open-ended natural language questions** about data
- You have invested in a **Genie space** with proper semantic layer configuration
- You are comfortable with a **flow-based polling loop** in Power Automate
- You do not need deterministic, structured return values

Use SQL Statement API instead when:
- You know the queries your agent will run ahead of time
- You need structured results for further processing
- You want simpler response parsing

---

## MCP (Model Context Protocol) Server

**Status: ✅ Supported (public endpoint) / ❌ Not supported (private endpoint without APIM)**

MCP is a protocol that allows AI models to call external tools in a structured way. Copilot Studio supports MCP as a first-class integration point for agents.

### How MCP Works with Copilot Studio

```
Copilot Studio Agent
    └── Discovers MCP tools via manifest at /.well-known/mcp.json (or equivalent)
    └── Calls individual tools via POST with structured input
    └── Receives structured tool response
    └── Incorporates result into agent reasoning
```

### The Key Distinction: MCP is not REST

The Databricks REST API is not an MCP server. To use MCP with Databricks:

1. You must **deploy an MCP server** (a separate service) that wraps Databricks APIs
2. The MCP server exposes **tools** (e.g., `run_sql_query`, `get_genie_answer`)
3. Copilot Studio calls the MCP server, not Databricks directly

This is a **product contract limitation** — not a networking or auth issue. Databricks does not ship an MCP server out-of-box.

### MCP Server Options

| Option | Status | Notes |
|---|---|---|
| Build a custom MCP server (Python/Node) | ✅ Supported | Full control; must be hosted and maintained |
| Use a community/open-source MCP server for Databricks | ⚠️ Works with caveats | Verify compatibility with Copilot Studio's MCP version |
| Use Databricks REST API directly as MCP | ❌ Not supported | Protocol mismatch |

### Endpoint Requirements for MCP

Copilot Studio requires MCP servers to be:
- Reachable over **HTTPS** (TLS required)
- Accessible from **Power Platform's outbound IPs** (no private endpoint)
- Responding with valid MCP manifest and tool definitions

See [03-mcp-servers.md](./03-mcp-servers.md) for deployment architecture.

---

## Decision Summary

```
What does your agent need to do?
│
├── Run a known SQL query and return results
│   └── Use: SQL Statement Execution API (via Custom Connector)
│
├── Answer natural language questions about data
│   ├── High query variability, semantic layer available → Use: Genie API
│   └── Known query patterns, structured output needed → Use: SQL Statement API
│
└── Call Databricks as a tool within a reasoning loop
    └── Use: MCP Server wrapping Databricks APIs
        └── Is the MCP server publicly accessible?
            ├── Yes → Connect directly from Copilot Studio
            └── No  → Requires APIM proxy (see 05-apim.md)
```
