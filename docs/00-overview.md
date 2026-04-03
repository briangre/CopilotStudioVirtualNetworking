# Copilot Studio → Azure Databricks: Architecture Decision Guide

## Who This Is For

Technical architects, Power Platform specialists, and developers who need to integrate Microsoft Copilot Studio agents with Azure Databricks. This guide assumes familiarity with Azure networking, OAuth 2.0, and REST APIs.

---

## Why This Document Exists

Connecting Copilot Studio to Databricks is not a single, clearly documented path. The combination of:

- Multiple Databricks endpoint types (SQL, Genie, MCP)
- Power Platform connector constraints
- Private networking requirements
- OAuth token acquisition inside Copilot Studio

…means architects routinely hit invisible walls. This guide surfaces those walls before you build.

---

## Supportability Matrix

| Approach | Works? | Key Constraint |
|---|---|---|
| Power Platform Databricks connector (out-of-box) | ✅ Supported | Public endpoint only; no Genie; limited auth options |
| Custom connector → Databricks SQL Statement API | ✅ Supported | Requires Entra ID OAuth; public or APIM-fronted endpoint |
| Custom connector → Databricks Genie API | ⚠️ Works with caveats | Genie is conversational, not tabular; output mapping is complex |
| MCP server exposed via public HTTPS | ✅ Supported | Copilot Studio supports MCP over HTTPS; server must handle auth |
| MCP server on private VNet (no APIM) | ❌ Not supported | Copilot Studio cannot reach private endpoints directly |
| MCP server via APIM (private backend) | ⚠️ Works with caveats | APIM must terminate TLS and handle auth relay; adds latency and cost |
| Direct Private Link from Power Platform | ❌ Not supported | Power Platform does not support outbound Private Link to arbitrary services |
| Databricks SQL Warehouse via JDBC/ODBC | ❌ Not supported | No connector exists for JDBC/ODBC in Copilot Studio action flows |
| Databricks REST API via HTTP action | ✅ Supported | Requires bearer token management; no built-in refresh handling |

---

## Choosing the Right Path

```
Is your Databricks workspace on a private VNet?
│
├── No (public endpoint available)
│   │
│   ├── Do you need conversational AI / Genie?
│   │   ├── Yes → Use Genie API via Custom Connector (see 02-endpoints.md)
│   │   └── No  → Use SQL Statement API via Custom Connector or Power Platform connector
│   │
│   └── Do you need tool/function calling (MCP)?
│       └── Yes → Expose MCP server publicly, call from Copilot Studio (see 03-mcp-servers.md)
│
└── Yes (private VNet)
    │
    ├── Can you place APIM in the VNet?
    │   ├── Yes → Use APIM as proxy; expose via public APIM endpoint (see 05-apim.md)
    │   └── No  → ❌ No supported path from Copilot Studio to private Databricks
    │
    └── Are you using MCP?
        └── Yes → MCP backend must be APIM-fronted or public (see 03-mcp-servers.md)
```

---

## Top Failure Modes

### 1. Assuming Private Link solves everything
Power Platform (including Copilot Studio) does not have an outbound Private Link capability to customer-managed VNets. If Databricks is private, you must proxy through APIM or another internet-accessible endpoint.

### 2. Confusing MCP protocol with REST
Copilot Studio's MCP integration expects HTTP + Server-Sent Events (SSE) or HTTP streaming. Standard Databricks REST APIs are not MCP servers. You must deploy an MCP-compatible server layer.

### 3. OAuth token acquisition in Copilot Studio
Copilot Studio does not natively acquire OAuth tokens mid-conversation for custom connectors the same way Logic Apps or Power Automate flows do. Understand where the token comes from before you build. See [Authentication Guide](./06-auth.md).

### 4. Genie output is not tabular
Genie returns conversational text responses, not structured result sets. If your agent needs to parse and present query results, use the SQL Statement API instead.

---

## Document Index

| Document | Topic |
|---|---|
| [01-connectors.md](./01-connectors.md) | Out-of-box Power Platform connectors for Databricks |
| [02-endpoints.md](./02-endpoints.md) | Choosing between Genie, SQL, and MCP endpoints |
| [03-mcp-servers.md](./03-mcp-servers.md) | MCP server architecture and Copilot Studio integration |
| [04-networking.md](./04-networking.md) | Public vs private networking considerations |
| [05-apim.md](./05-apim.md) | Using Azure API Management as a proxy layer |
| [06-auth.md](./06-auth.md) | Entra ID app registration and OAuth flows |
