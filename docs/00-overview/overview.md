# Overview

This documentation set describes how to connect **Copilot Studio agents** to **Azure Databricks** MCP endpoints (`mcpsql` and `mcpgenie`) across a range of connectivity and authentication architectures.

## Purpose

Rather than writing one end-to-end guide per configuration, this set uses a **components → patterns → config-paths** model:

| Layer | Folder | Description |
|-------|--------|-------------|
| Components | [`/10-components`](../10-components/) | Atomic, reusable building blocks |
| Patterns | [`/20-patterns`](../20-patterns/) | Composable architecture patterns |
| Config Paths | [`/30-config-paths`](../30-config-paths/) | Thin assembly guides for specific scenarios |
| Decision Guides | [`/40-decision-guides`](../40-decision-guides/) | Flowcharts to pick the right config |
| Matrix | [`/50-matrix`](../50-matrix/) | Configuration compatibility matrix |
| Reference | [`/99-reference`](../99-reference/) | Glossary and links |

## Architecture Dimensions

Each configuration is defined by choices along four dimensions:

1. **Copilot Studio agent** – the Power Platform surface used to connect to Databricks
2. **Connector type** – Out-of-box (OOB) vs. Custom connector
3. **Network path** – Public (internet-routable) vs. Private (VNet/Private Endpoint)
4. **Databricks endpoint** – `mcpsql` (SQL warehouse) vs. `mcpgenie` (AI/BI Genie)

APIM is an optional proxy layer applicable to the private network path.

## Quick Start

→ Not sure where to start? Use the [decision guides](../40-decision-guides/decision-which-connector.md).  
→ Need to compare options at a glance? See the [configuration matrix](../50-matrix/matrix-configurations.md).  
→ Understand how to read these docs: [How to Use This Docs](how-to-use-this-docs.md).

## Scope

- **In scope**: Connectivity, authentication, and network configuration for Copilot Studio agents ↔ Databricks.
- **Out of scope**: Databricks workspace provisioning, Power Platform environment setup, Azure subscription management (except where directly referenced).
