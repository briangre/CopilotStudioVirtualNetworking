# Component: APIM Private Ingress and Egress

## Purpose

Describes how Azure API Management (APIM) is deployed in a private network topology to act as a secure gateway between Power Platform and Databricks, handling both inbound (ingress) traffic from Power Platform and outbound (egress) traffic to Databricks.

## When to Use

Reference this component when:
- You require a private, VNet-integrated API gateway in front of Databricks.
- You need centralized policy enforcement (rate limiting, JWT validation, logging) for multiple Power Platform consumers.
- Traffic from Power Platform must not traverse the public internet.

## Inputs / Prerequisites

- Azure VNet with at least two subnets: one for APIM, one for the Databricks Private Endpoint.
- APIM instance deployed in **Internal** VNet integration mode (recommended for fully private topology).
- Private Endpoint for the Databricks workspace reachable from the APIM subnet.
- DNS resolution configured so APIM can resolve the Databricks private endpoint hostname.
- [Power Platform Virtual Networking](https://learn.microsoft.com/en-us/power-platform/admin/virtual-network-support-overview) configured for Power Platform egress to reach APIM's internal IP.

## Outputs / What "Done" Looks Like

- Power Platform connector calls APIM's internal or private frontend IP.
- APIM forwards the request to the Databricks Private Endpoint.
- Databricks public network access can be fully disabled.
- APIM logs show successful proxied calls.

## Configuration Steps

1. [Deploy APIM in VNet-integrated mode](https://learn.microsoft.com/en-us/azure/api-management/api-management-using-with-internal-vnet) (Internal mode is recommended for a fully private topology).
2. [Configure APIM's backend in its own virtual network](https://learn.microsoft.com/en-us/azure/api-management/private-endpoint) so that the Databricks hostname resolves to its private IP address via the Databricks Private Endpoint.
3. Create an API using **Expose existing MCP** to front the Databricks MCP endpoint.
4. With successful [Power Platform Virtual Networking configuration](../../10-components/power-platform/component-pp-networking.md), configure the Copilot Studio Agent MCP as a tool (see [component-apim-mcp-proxy-basics](component-apim-mcp-proxy-basics.md), which creates the custom connector).
5. Test end-to-end communication before applying policies.
6. Apply desired inbound policies (e.g., JWT validation, rate limiting, subscription key or OAuth enforcement) and outbound policies as needed for response transformation.

> **Note on authentication**: Authentication to Databricks is handled automatically by the OAuth flow configured in the connector — no APIM policy is required to inject or manage Databricks auth tokens.

## Validation Steps

1. From a VM in the APIM Inbound VNet, `curl` the APIM frontend URL — expect HTTP 200 or 401.
2. Execute a test call through the Power Platform connector — expect HTTP 200.
3. Check APIM diagnostic logs to confirm the request was proxied to Databricks.

## Known Limitations

- APIM deployment can take 30–45 minutes.
- APIM Premium SKU required for multi-region or zone-redundant deployments.

## Related

- [component-apim-mcp-proxy-basics](component-apim-mcp-proxy-basics.md)
- [component-public-vs-private](../networking/component-public-vs-private.md)
- [pattern-private-via-apim](../../20-patterns/connectivity/pattern-private-via-apim.md)
