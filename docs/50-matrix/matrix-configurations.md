# Configuration Matrix

This matrix maps every validated configuration combination to its config path.

> Legend: ✅ Supported · ⚠️ Limited/partial · ❌ Not supported · 🔲 TODO: confirm

| # | Power Platform Surface | Connector Type | Network Path | APIM | Databricks Endpoint | Config Path |
|---|------------------------|---------------|--------------|------|---------------------|-------------|
| 01 | Copilot Studio | OOB | Public | No | mcpgenie | [config-01](../30-config-paths/config-01-pp-oob-public-dbx-genie.md) |
| 02 | Copilot Studio | Custom | Private | Yes | mcpsql | [config-02](../30-config-paths/config-02-pp-custom-private-apim-dbx-sql.md) |
| 03 | Copilot Studio | OOB | Private | No | mcpgenie | [config-03](../30-config-paths/config-03-pp-oob-private-dbx-genie.md) |
| 04 | Power Automate | OOB | Public | No | mcpgenie | 🔲 TODO: confirm |
| 05 | Power Automate | Custom | Private | Yes | mcpsql | 🔲 TODO: confirm |
| 06 | Power Apps | Custom | Private | Yes | mcpsql | 🔲 TODO: confirm |

## Dimension Definitions

### Power Platform Surface
- **Copilot Studio** – Conversational agent with MCP tool calling.
- **Power Automate** – Automated flow invoking Databricks via connector.
- **Power Apps** – Canvas or model-driven app making direct connector calls.

### Connector Type
- **OOB (Out-of-Box)** – Pre-built connector available in the Power Platform connector gallery. No custom code required. See [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md).
- **Custom** – Developer-built connector using an OpenAPI spec. Required when OOB connector capabilities are insufficient. See [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md).

### Network Path
- **Public** – Traffic flows over the public internet; Databricks endpoint is internet-accessible.
- **Private** – Traffic stays within Azure VNet via Private Endpoints. See [component-public-vs-private](../10-components/networking/component-public-vs-private.md).

### APIM
- **Yes** – Azure API Management acts as a proxy/gateway in the request path. See [component-apim-mcp-proxy-basics](../10-components/apim/component-apim-mcp-proxy-basics.md).
- **No** – Direct connection from connector to Databricks endpoint.

### Databricks Endpoint
- **mcpsql** – MCP endpoint backed by a SQL warehouse. See [component-dbx-mcpsql](../10-components/databricks/component-dbx-mcpsql.md).
- **mcpgenie** – MCP endpoint backed by AI/BI Genie. See [component-dbx-mcpgenie](../10-components/databricks/component-dbx-mcpgenie.md).
