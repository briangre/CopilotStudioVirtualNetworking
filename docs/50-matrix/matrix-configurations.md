# Configuration Matrix

This matrix maps every validated configuration combination to its config path. Configurations are grouped by network path, then by whether APIM is used.

> Legend: ✅ Supported · ⚠️ Limited/partial · ❌ Not supported · 🔲 TODO: confirm

## Public Network (no private networking required)

| # | Power Platform Surface | Connector Type | Network Path | APIM | Databricks Endpoint | Config Path |
|---|------------------------|---------------|--------------|------|---------------------|-------------|
| 01 | Copilot Studio | OOB | Public | No | mcpgenie | [config-01](../30-config-paths/config-01-pp-oob-public-dbx-genie.md) |
| 05 | Copilot Studio | Custom | Public | No | mcpsql | [config-05](../30-config-paths/config-05-pp-custom-public-dbx-sql.md) |

## Private Network · Direct (no APIM)

| # | Power Platform Surface | Connector Type | Network Path | APIM | Databricks Endpoint | Config Path |
|---|------------------------|---------------|--------------|------|---------------------|-------------|
| 03 | Copilot Studio | OOB | Private | No | mcpgenie | [config-03](../30-config-paths/config-03-pp-oob-private-dbx-genie.md) |

## Private Network · Via APIM

| # | Power Platform Surface | Connector Type | Network Path | APIM | Databricks Endpoint | Config Path |
|---|------------------------|---------------|--------------|------|---------------------|-------------|
| 02 | Copilot Studio | Custom | Private | Yes | mcpsql | [config-02](../30-config-paths/config-02-pp-custom-private-apim-dbx-sql.md) |
| 04 | Copilot Studio | Custom | Private | Yes | mcpgenie | [config-04](../30-config-paths/config-04-pp-custom-private-apim-dbx-genie.md) |

## Dimension Definitions

### Power Platform Surface
- **Copilot Studio** – Conversational agent with MCP tool calling.

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
