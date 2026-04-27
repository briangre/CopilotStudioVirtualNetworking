# Overview

This documentation set describes how to connect **Copilot Studio agents** to **backend data services** over **private Azure networking**, using a pluggable architecture that accommodates multiple data service targets, connectivity models, and authentication approaches.

## Purpose

Rather than writing one end-to-end guide per configuration, this set uses a **components → patterns → config-paths** model:

| Layer | Folder | Description |
|-------|--------|-------------|
| Components | [`/10-components`](../10-components/) | Atomic, reusable building blocks |
| Patterns | [`/20-patterns`](../20-patterns/) | Composable architecture patterns |
| Config Paths | [`/30-config-paths`](../30-config-paths/) | Thin assembly guides for specific scenarios |
| Decision Guides | [`/40-decision-guides`](../40-decision-guides/) | Flowcharts to pick the right config |
| Matrix | [`/50-matrix`](../50-matrix/) | Configuration compatibility matrix |

## Architecture Dimensions

Each configuration is defined by choices along four dimensions:

1. **Copilot Studio agent** – the Power Platform surface making the outbound connection
2. **Connector type** – Out-of-box (OOB) vs. Custom connector
3. **Network path** – Public (internet-routable) vs. Private (VNet/Private Endpoint)
4. **Target data service endpoint** – the specific service and endpoint being reached (e.g., a SQL endpoint, an AI/BI query interface, or another MCP-compatible service)

APIM is an optional proxy layer applicable to provide additional logging and filtering over either the 
private or public network paths.

## Quick Start

→ Not sure where to start? Use the [decision guides](../40-decision-guides/).  
→ Need to compare options at a glance? See the [configuration matrix](../50-matrix/matrix-configurations.md).  
→ Understand how to read these docs: [How to Use This Docs](how-to-use-this-docs.md).

## Scope

- **In scope**: Connectivity, authentication, and network configuration for Copilot Studio agents connecting to backend data services over private Azure networking.
- **Out of scope**: Data service-specific provisioning, Power Platform environment setup, Azure subscription management (except where directly referenced).
