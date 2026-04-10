# Component: Power Platform Networking

## Purpose

Describes and compares the networking options available for Copilot Studio agents when making outbound calls to external APIs such as Azure Databricks MCP endpoints. The goal is to help you choose the right path for your scenario — not to provide step-by-step configuration instructions.

## When to Use

Reference this component when:
- Selecting a network path for Power Platform egress to Databricks.
- Understanding the trade-offs between public connectivity, On-Premises Data Gateway (OPDG), and Virtual Network (VNet) integration.
- Discussing requirements with your network or security team before committing to an approach.

---

## Overview of Networking Options

Power Platform (including Copilot Studio) offers three broad options for outbound connectivity to external APIs such as Databricks MCP endpoints:

| Option | Connectivity Model | Infrastructure Required | Complexity | Best For |
|---|---|---|---|---|
| **Public** | Traffic travels over the public internet (TLS encrypted) | None beyond existing Databricks public endpoint | Low | Prototypes, dev/test, or environments where public internet access is acceptable |
| **Private via On-Premises Data Gateway (OPDG)** | Traffic is routed through a self-hosted gateway agent running on a Windows VM in your network | Windows Server VM with network line-of-sight to Databricks | Medium | ❌ Not applicable for Databricks — OOB connector not supported over gateway; custom connectors cannot use OAuth over gateway |
| **Private via Virtual Network (VNet) Integration** | Power Platform egress is bound to enterprise-managed Azure VNets using Enterprise Policy | Azure VNets in paired regions, delegated subnets, Enterprise Policy, managed environment | High | Enterprise scenarios requiring network-layer isolation and a cloud-native, VM-free approach |

---

## Option 1: Public Network

In the public networking model, Copilot Studio connectors communicate with Databricks MCP endpoints directly over the internet. Connections are protected by TLS and OAuth tokens, but there is no network-layer isolation — traffic is not constrained to private IP space.

**When to choose this option:**
- Your security policy permits internet-routed traffic for this workload.
- The Databricks workspace has public network access enabled.
- You need the fastest path to a working proof-of-concept or development environment.
- Your organization does not yet have Azure VNet infrastructure aligned with Power Platform regions.

**Key considerations:**
- Power Platform makes outbound calls from a shared pool of IP ranges. You can optionally add these IP ranges to a Databricks network access list to restrict inbound connections to Power Platform only. See [Power Platform outbound IP addresses](https://learn.microsoft.com/en-us/power-platform/admin/online-requirements) for the current list.
- There is no guarantee of static outbound IP addresses on the public path. IP allow-listing provides some assurance but is not a substitute for network-layer isolation.
- This option cannot be used if the Databricks workspace has public network access disabled.

---

## Option 2: Private via On-Premises Data Gateway (OPDG)

> ⚠️ **Not applicable for Databricks scenarios.** OPDG is listed here for completeness, but it cannot be used to connect Power Platform to Azure Databricks MCP endpoints. See the limitations below.

The On-Premises Data Gateway is a self-hosted agent that runs on a Windows Server VM within your network. Power Platform routes connector traffic through this gateway, which then forwards requests to on-premises or privately networked resources.

**Why OPDG does not work for Databricks:**
- The out-of-box (OOB) Databricks connector for Power Platform does not support routing through an On-Premises Data Gateway.
- Custom connectors in Power Platform cannot use OAuth authentication when routed through the On-Premises Data Gateway. Since Databricks MCP endpoints require OAuth (via Entra ID / service principal), this rules out OPDG for any custom connector approach as well.

For private connectivity to Databricks from Power Platform, use **Option 3: VNet Integration** instead.

---

## Option 3: Private via Virtual Network (VNet) Integration

Power Platform Virtual Network integration (sometimes called VNet injection or enterprise networking) binds Power Platform egress traffic to Azure-managed VNets. This is done by associating Power Platform environments with an Enterprise Policy that references subnets delegated to `Microsoft.PowerPlatform/enterprisePolicies`. All outbound connector traffic from those environments then flows through those VNets, enabling private connectivity without a gateway VM.

**When to choose this option:**
- Your organization requires a cloud-native, VM-free private connectivity solution.
- You need Power Platform to participate in the same private network as Databricks (via VNet peering to the Databricks VNet).
- Your security architecture requires all traffic between Power Platform and Databricks to remain on private IP space.
- You have (or are willing to provision) managed Power Platform environments and the appropriate Azure VNet infrastructure in [supported region pairs](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#supported-regions).

**Key considerations:**
- VNet integration requires a **managed Power Platform environment**, which has licensing implications. Confirm with your licensing team before selecting this path.
- Two Azure VNets must be provisioned — one in each of the required paired regions (e.g., East US / West US). Each VNet must have at least one subnet delegated to `Microsoft.PowerPlatform/enterprisePolicies`.
- The Power Platform VNets must be peered with the VNet hosting the Databricks Private Endpoint, and the private DNS zone for `privatelink.azuredatabricks.net` must be linked to the Power Platform VNets.
- This is the most complex option to set up but produces the cleanest enterprise architecture — no VMs, no gateway agents, and full network-layer isolation.
- For setup details, refer to: [Set up virtual network support for Power Platform](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-setup-configure)

---

## Choosing the Right Option

For Databricks scenarios, the choice is straightforward — OPDG is not viable (see Option 2 above), so the decision is between public and VNet integration:

1. **Does your security policy require private network connectivity (no public internet)?**
   - **No** → Public networking is sufficient. Start here for development or testing.
   - **Yes** → VNet Integration (Option 3) is the only supported private path for Databricks.

2. **Do you have (or can you obtain) managed Power Platform environments and the required Azure VNet infrastructure in supported paired regions?**
   - **Yes** → Proceed with VNet integration.
   - **No** → Work with your platform team to provision the prerequisites. There is no OPDG fallback for Databricks connectivity.

---

## Related

- [component-public-vs-private](../networking/component-public-vs-private.md)
- [decision-public-vs-private](../../40-decision-guides/decision-public-vs-private.md)
- [pattern-public-direct](../../20-patterns/connectivity/pattern-public-direct.md)
- [pattern-private-end-to-end](../../20-patterns/connectivity/pattern-private-end-to-end.md)
