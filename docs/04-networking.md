# Networking: Public vs Private Databricks Workspaces

## The Core Problem

Power Platform, including Copilot Studio, initiates outbound HTTPS connections from Microsoft-managed infrastructure. It does not support:

- Outbound connections through customer-managed Private Link endpoints
- Injection into customer VNets
- Direct connectivity to services that only accept traffic from specific private subnets

This is a **platform boundary**, not a configuration limitation.

---

## Networking Scenarios

### Scenario 1: Databricks with Public Endpoint

**Status: ✅ Fully supported**

```
Copilot Studio / Power Platform
    │
    │  HTTPS (internet)
    ▼
Databricks Workspace
(public endpoint enabled, firewall allows Power Platform IPs)
```

Requirements:
- Databricks workspace has public endpoint enabled
- Network Security Group / IP allowlist permits Power Platform's [published outbound IP ranges](https://learn.microsoft.com/en-us/connectors/common/outbound-ip-addresses)
- Databricks Personal Access Token or Entra ID OAuth configured

This is the simplest path. If your security posture permits a public endpoint, use this.

---

### Scenario 2: Databricks Private Workspace — No Proxy

**Status: ❌ Not supported**

```
Copilot Studio / Power Platform
    │
    │  HTTPS (internet)
    ▼
    ╳  (Databricks has no public endpoint)
Databricks Workspace (private VNet only)
```

There is no workaround to make this work without introducing a proxy layer. This is a hard boundary:

- Power Platform cannot join your VNet
- Databricks private endpoint is not reachable from Microsoft-managed infrastructure
- VNet peering with Power Platform's internal network is not possible for customers

**Do not attempt to work around this with:**
- Firewall exceptions (the connection is blocked at network level, not app level)
- DNS tricks (the IP won't route)
- Changing Databricks auth settings (auth is not the issue here)

---

### Scenario 3: Databricks Private Workspace — APIM Proxy

**Status: ⚠️ Supported with additional architecture**

```
Copilot Studio / Power Platform
    │
    │  HTTPS (internet)
    ▼
Azure API Management
(public-facing tier, e.g., Developer or Standard v2)
    │
    │  Internal VNet routing / Private Link
    ▼
Databricks Workspace (private endpoint)
```

APIM bridges the gap between public Power Platform and private Databricks. See [05-apim.md](./05-apim.md) for configuration.

**Key requirements for this pattern:**
- APIM must be in a tier that supports VNet integration (Developer, Premium, or Standard v2)
- APIM must be able to reach the Databricks private endpoint (VNet peering or APIM VNet injection)
- APIM exposes a public HTTPS endpoint that Power Platform can call
- Authentication flows through APIM policies (token validation, forwarding)

---

### Scenario 4: Databricks with Both Public and Private Endpoints

**Status: ✅ Supported (use public endpoint path)**

Some workspaces have both endpoints enabled for migration or hybrid purposes. In this case:

- Point Power Platform at the **public endpoint**
- Apply IP allowlisting to restrict which sources can use the public endpoint
- Use the private endpoint for internal VNet traffic (other Azure services, Spark clusters)

This is operationally sound if your security team permits it.

---

## Power Platform Outbound IP Ranges

Power Platform uses documented outbound IP ranges per region. When allowlisting Databricks firewall rules, you must include:

- The IP ranges for your **Power Platform environment's region**
- These are available at: https://learn.microsoft.com/en-us/connectors/common/outbound-ip-addresses

> ⚠️ These IP ranges are shared across all Power Platform tenants in the region. Allowlisting them does not restrict access to your tenant only. Combine with **Entra ID authentication** to enforce tenant-level access control.

---

## Databricks Workspace Firewall Configuration

When using a public endpoint with IP allowlisting:

1. In the Databricks workspace, go to **Settings → Networking**
2. Enable **IP access list**
3. Add the Power Platform outbound IP ranges for your region
4. Optionally add APIM's outbound IPs if routing through APIM

> ⚠️ IP allowlists in Databricks apply to the entire workspace. Ensure you include any other services (CI/CD, monitoring, developer machines) before enabling.

---

## What About Azure Private Link for Power Platform?

Power Platform supports **inbound** Private Link for some scenarios (e.g., connecting on-premises services via ExpressRoute or VPN, or using the On-Premises Data Gateway).

However, **outbound** Private Link — where Power Platform initiates a connection to a private endpoint in your VNet — is not supported for custom connectors or Copilot Studio MCP integrations.

The On-Premises Data Gateway is a potential workaround for some Power Automate scenarios, but:
- It requires a VM running the gateway agent inside your VNet
- It adds operational overhead (patching, availability, capacity)
- It does not support all connector types
- It is not directly usable for Copilot Studio HTTP actions or MCP

This is a viable option only for specific Power Automate-based integrations, not for Copilot Studio agent actions directly.

---

## Networking Decision Summary

| Your Situation | Recommended Path |
|---|---|
| Databricks has public endpoint | Connect directly; use IP allowlist + Entra ID OAuth |
| Databricks is fully private, APIM available | Route through APIM (see [05-apim.md](./05-apim.md)) |
| Databricks is fully private, no APIM | ❌ No supported path from Copilot Studio |
| Both endpoints enabled | Use public endpoint; tighten with IP allowlist |
| Power Automate flow (not Copilot Studio direct) | On-premises data gateway may be viable (limited) |
