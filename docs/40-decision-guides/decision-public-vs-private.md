# Decision Guide: Public Network vs. Private Network

Use this guide to decide whether to route Power Platform ↔ Databricks traffic over the **public internet** or through a **private (VNet) network path**.

## Decision Tree

```
Start
  │
  ▼
Does your organization's security policy prohibit
data traversing the public internet?
  │
  ├─ YES ──► Private network path required.
  │          Do you already have an Azure VNet with
  │          Private Endpoints for Databricks?
  │            │
  │            ├─ YES ──► Do you need centralized policy
  │            │          enforcement / API transformation?
  │            │            │
  │            │            ├─ YES ──► Private via APIM
  │            │            │         pattern-private-via-apim
  │            │            │
  │            │            └─ NO  ──► Private end-to-end
  │            │                       pattern-private-end-to-end
  │            │
  │            └─ NO  ──► Provision VNet + Private Endpoint first.
  │                       See component-public-vs-private (prereqs).
  │
  └─ NO  ──► Does your Databricks workspace allow
             public network access?
               │
               ├─ YES ──► Public direct pattern
               │          pattern-public-direct
               │
               └─ NO  ──► Private network path required (see above).
```

## Public Network Path

**Use when:**
- Security policy permits public internet traffic for this workload.
- Databricks workspace has public network access enabled.
- Quickest path to a working prototype.

**Trade-offs:**
- Traffic transits the public internet (mitigated by TLS + OAuth tokens).
- Databricks workspace firewall must permit inbound from Power Platform IPs. TODO: confirm exact IP ranges.
- No network-layer isolation.

→ [pattern-public-direct](../20-patterns/connectivity/pattern-public-direct.md)  
→ [component-public-vs-private](../10-components/networking/component-public-vs-private.md)

## Private Network Path (without APIM)

**Use when:**
- End-to-end private connectivity is required.
- No need for centralized policy enforcement or transformation at the gateway layer.

**Trade-offs:**
- Requires Azure VNet, Private Endpoint for Databricks, and on-premises data gateway (OPDG) or VNet data gateway for Power Platform egress.
- Higher setup complexity.

→ [pattern-private-end-to-end](../20-patterns/connectivity/pattern-private-end-to-end.md)

## Private Network Path (via APIM)

**Use when:**
- Private connectivity is required **and** you need centralized rate limiting, policy enforcement, or protocol translation.
- Multiple Copilot Studio agents share a single gateway.

**Trade-offs:**
- APIM adds cost and operational overhead.
- APIM must itself be deployed in or integrated with the same VNet.

→ [pattern-private-via-apim](../20-patterns/connectivity/pattern-private-via-apim.md)  
→ [component-apim-private-ingress-egress](../10-components/apim/component-apim-private-ingress-egress.md)

## Related

- [decision-which-connector](decision-which-connector.md)
- [matrix-configurations](../50-matrix/matrix-configurations.md)
