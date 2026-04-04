# Component: Power Platform Networking

## Purpose

Describes the networking options available for Power Platform services (Copilot Studio, Power Automate, Power Apps) when making outbound calls to external APIs such as Azure Databricks MCP endpoints.

## When to Use

Reference this component when:
- Selecting a network path (public vs. private) for Power Platform egress.
- Configuring an on-premises data gateway (OPDG) or VNet data gateway.
- Troubleshooting connectivity failures from Power Platform to Databricks.

## Inputs / Prerequisites

- Power Platform environment provisioned in a region that supports VNet data gateways (if private path required). TODO: confirm supported regions.
- Azure subscription with permissions to create Virtual Networks, Private Endpoints, and Data Gateways.
- Databricks workspace URL and network configuration known.

## Outputs / What "Done" Looks Like

- Power Platform connector can successfully invoke the target Databricks endpoint.
- No timeouts or TLS errors in connector test calls.
- Outbound IP address (public path) or gateway resource (private path) is documented and whitelisted if required.

## Configuration Steps

### Public Network Path

1. Verify Databricks workspace has public network access enabled.
2. Note the outbound IP ranges for your Power Platform region. TODO: confirm IP range documentation link.
3. Add those IPs to the Databricks workspace network access list if IP-based allow-listing is used.
4. Test the connector (see Validation below).

### Private Network Path (VNet Data Gateway)

1. Deploy an Azure VNet data gateway in the same VNet as the Databricks Private Endpoint.
2. Register the VNet data gateway in the Power Platform admin center.
3. Configure the connector to use the VNet data gateway.
4. Verify that the gateway has line-of-sight to the Databricks Private Endpoint DNS name.

## Validation Steps

1. In Power Automate or Power Apps, create a test flow/action using the connector.
2. Invoke a lightweight call (e.g., list tables or health check).
3. Confirm HTTP 200 response within expected latency.
4. Check gateway logs (if using VNet data gateway) for any errors.

## Known Limitations

- VNet data gateway is not available in all Power Platform plans. TODO: confirm plan requirements.
- OPDG requires a Windows VM; VNet data gateway is preferred for cloud-native setups.
- Static outbound IP allocation is not guaranteed on the public path.

## Related

- [component-public-vs-private](../networking/component-public-vs-private.md)
- [pattern-public-direct](../../20-patterns/connectivity/pattern-public-direct.md)
- [pattern-private-end-to-end](../../20-patterns/connectivity/pattern-private-end-to-end.md)
