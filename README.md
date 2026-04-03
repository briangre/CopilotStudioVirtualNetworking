# Copilot Studio → Azure Databricks

Decision-driven documentation for architects and makers connecting Microsoft Copilot Studio agents to Azure Databricks.

## What This Covers

This guide helps you make the correct architectural decision for your integration scenario — and understand why certain paths fail — across:

- Out-of-box Power Platform connectors
- MCP (Model Context Protocol) servers
- Genie vs SQL vs MCP endpoint selection
- Public vs private networking
- Azure API Management as a proxy layer
- Entra ID app registration and OAuth flows

## Documentation

| Document | Description |
|---|---|
| [00-overview.md](./docs/00-overview.md) | Start here. Supportability matrix and decision tree for all integration paths |
| [01-connectors.md](./docs/01-connectors.md) | Out-of-box Power Platform connectors: what they support and where they fall short |
| [02-endpoints.md](./docs/02-endpoints.md) | Choosing between Genie, SQL Statement, and MCP endpoints |
| [03-mcp-servers.md](./docs/03-mcp-servers.md) | MCP server architecture, hosting options, and Copilot Studio integration |
| [04-networking.md](./docs/04-networking.md) | Public vs private networking: what Power Platform can and cannot reach |
| [05-apim.md](./docs/05-apim.md) | Using Azure API Management to bridge Copilot Studio and private Databricks |
| [06-auth.md](./docs/06-auth.md) | Entra ID app registrations, OAuth flows, and token acquisition |

## Quick Start: Which Path Is Right For Me?

**Start with [00-overview.md](./docs/00-overview.md)** — it includes a full supportability matrix and a decision tree that will route you to the right document for your scenario.

## Supportability Labels

Throughout this documentation:

- ✅ **Supported** — Works as documented; suitable for production
- ⚠️ **Works with caveats** — Functional but requires additional design consideration
- ❌ **Not supported** — Does not work; a product or protocol limitation, not a configuration issue
