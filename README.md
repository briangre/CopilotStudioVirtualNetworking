# Copilot Studio → Azure Databricks

Componentized documentation for connecting **Copilot Studio agents** to **Azure Databricks** MCP endpoints (`mcpsql`, `mcpgenie`) across multiple connectivity and authentication architectures.

## How to Navigate

This documentation uses a **components → patterns → config-paths** model to avoid duplicating content across configurations.

| Layer | Folder | Purpose |
|-------|--------|---------|
| Overview | [docs/00-overview](./docs/00-overview/) | Start here |
| Components | [docs/10-components](./docs/10-components/) | Atomic building blocks |
| Patterns | [docs/20-patterns](./docs/20-patterns/) | Composable architecture designs |
| Config Paths | [docs/30-config-paths](./docs/30-config-paths/) | Step-by-step scenario guides |
| Decision Guides | [docs/40-decision-guides](./docs/40-decision-guides/) | Choose your path |
| Matrix | [docs/50-matrix](./docs/50-matrix/) | All configurations at a glance |
| Reference | [docs/99-reference](./docs/99-reference/) | Glossary and links |

## Quick Start

1. **Not sure where to start?** → [docs/40-decision-guides/decision-which-connector.md](./docs/40-decision-guides/decision-which-connector.md)
2. **Want to compare all options?** → [docs/50-matrix/matrix-configurations.md](./docs/50-matrix/matrix-configurations.md)
3. **Understand the doc structure** → [docs/00-overview/how-to-use-this-docs.md](./docs/00-overview/how-to-use-this-docs.md)

## Possible Configurations

| # | Connector | Network | APIM | Endpoint | Config Path |
|---|-----------|---------|------|----------|-------------|
| 01 | OOB | Public | No | mcpgenie | [config-01](./docs/30-config-paths/config-01-pp-oob-public-dbx-genie.md) |
| 02 | Custom | Private | Yes | mcpsql | [config-02](./docs/30-config-paths/config-02-pp-custom-private-apim-dbx-sql.md) |
| 03 | OOB | Private | No | mcpgenie | [config-03](./docs/30-config-paths/config-03-pp-oob-private-dbx-genie.md) |
| 04 | Custom | Private | Yes | mcpgenie | [config-04](./docs/30-config-paths/config-04-pp-custom-private-apim-dbx-genie.md) |
| 05 | Custom | Public | No | mcpsql | [config-05](./docs/30-config-paths/config-05-pp-custom-public-dbx-sql.md) |
| 06 | Custom | Private | No | mcpsql | [config-06](./docs/30-config-paths/config-06-pp-custom-private-dbx-sql.md) |

## Notation

- ✅ Supported · ⚠️ Limited/partial · ❌ Not supported · 🔲 TODO: confirm
