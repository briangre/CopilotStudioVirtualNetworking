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
- APIM instance deployed in **Internal** or **External** VNet integration mode. TODO: confirm recommended mode.
- Private Endpoint for the Databricks workspace reachable from the APIM subnet.
- DNS resolution configured so APIM can resolve the Databricks private endpoint hostname.
- VNet data gateway (or OPDG) for Power Platform egress, with line-of-sight to APIM's internal IP.

## Outputs / What "Done" Looks Like

- Power Platform connector calls APIM's internal or private frontend IP.
- APIM forwards the request to the Databricks Private Endpoint.
- Databricks public network access can be fully disabled.
- APIM logs show successful proxied calls.

## Configuration Steps

1. Deploy APIM in VNet-integrated mode (Internal recommended for fully private).
2. Configure APIM's backend to point to the Databricks private endpoint URL.
3. Create an APIM API (see [component-apim-mcp-proxy-basics](component-apim-mcp-proxy-basics.md)) for the MCP endpoint.
4. Deploy a VNet data gateway in the same VNet (or peered VNet with routing to APIM).
5. Configure the Power Platform custom connector to call APIM's frontend URL.
6. Apply inbound policies: JWT validation, rate limiting, subscription key or OAuth enforcement.
7. Apply outbound policies: append Databricks auth token, transform response if needed.

## Validation Steps

1. From a VM in the VNet, `curl` the APIM frontend URL — expect HTTP 200 or 401.
2. From the VNet data gateway host, confirm APIM frontend URL is reachable.
3. Execute a test call through the Power Platform connector — expect HTTP 200.
4. Check APIM diagnostic logs to confirm the request was proxied to Databricks.
5. Disable Databricks public network access and re-test to confirm end-to-end private path.

## Known Limitations

- APIM Internal VNet mode requires a custom DNS setup for external management plane access. TODO: confirm DNS requirements.
- APIM deployment can take 30–45 minutes.
- APIM does not automatically rotate Databricks credentials; secret refresh must be scripted. TODO: confirm recommended approach.
- APIM Premium SKU required for multi-region or zone-redundant deployments.

## Related

- [component-apim-mcp-proxy-basics](component-apim-mcp-proxy-basics.md)
- [component-public-vs-private](../networking/component-public-vs-private.md)
- [pattern-private-via-apim](../../20-patterns/connectivity/pattern-private-via-apim.md)
