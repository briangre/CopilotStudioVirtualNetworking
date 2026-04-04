# Component: Public vs. Private Network

## Purpose

Describes the two network path options for Power Platform ↔ Databricks traffic, their infrastructure requirements, and how to choose between them.

## When to Use

Reference this component when:
- Designing the network architecture for a new Power Platform ↔ Databricks integration.
- Determining whether a Private Endpoint and VNet data gateway are required.
- Troubleshooting network-level connectivity failures.

## Inputs / Prerequisites

### Public Path
- Databricks workspace with public network access enabled.
- Power Platform outbound IP ranges documented and, if needed, added to Databricks network allow-list. TODO: confirm IP range documentation.

### Private Path
- Azure VNet in the same region as the Databricks workspace.
- Private Endpoint for the Databricks workspace deployed into a subnet of that VNet.
- Private DNS zone configured for `*.azuredatabricks.net` resolving to the Private Endpoint IP.
- VNet data gateway (or OPDG) deployed in the same VNet, registered in the Power Platform admin center.

## Outputs / What "Done" Looks Like

- **Public**: Connector test returns HTTP 200; traffic is routed over the internet.
- **Private**: Connector test returns HTTP 200; Databricks workspace's public network access can be disabled; DNS resolves to a private IP.

## Architecture Summary

### Public Path

```
Power Platform ──[internet]──► Databricks Public Endpoint
```

### Private Path

```
Power Platform ──[VNet DGW]──[Azure VNet]──[Private Endpoint]──► Databricks
```

## Configuration Steps

### Public Path

1. Confirm Databricks workspace "Public network access" is `Enabled`.
2. Optionally add Power Platform IP ranges to the Databricks IP access list.
3. Configure the connector with the public workspace URL.

### Private Path

1. Create a VNet (or use an existing one in the same region as Databricks).
2. Deploy a Private Endpoint for the Databricks workspace into a dedicated subnet.
3. Configure the Private DNS zone `privatelink.azuredatabricks.net` and link it to the VNet.
4. Deploy a VNet data gateway in the same VNet.
5. Register the gateway in the Power Platform admin center.
6. Configure the connector to route through the VNet data gateway.
7. Optionally disable public network access on the Databricks workspace.

## Validation Steps

1. From a VM inside the VNet, run `nslookup <workspace>.azuredatabricks.net` — expect a private IP.
2. From the same VM, `curl https://<workspace>.azuredatabricks.net/api/2.0/clusters/list` — expect HTTP 200 or 401 (not a network error).
3. Test the Power Platform connector through the VNet data gateway.
4. Disable public network access on Databricks (if desired) and re-test to confirm private-only access works.

## Known Limitations

- VNet data gateway requires a Premium Power Platform license or pay-per-use. TODO: confirm license requirement.
- Private Endpoint DNS resolution can take several minutes to propagate.
- When public access is disabled on Databricks, all management tools (Azure portal, Terraform) must also use the private path.

## Related

- [component-pp-networking](../power-platform/component-pp-networking.md)
- [component-apim-private-ingress-egress](../apim/component-apim-private-ingress-egress.md)
- [pattern-public-direct](../../20-patterns/connectivity/pattern-public-direct.md)
- [pattern-private-end-to-end](../../20-patterns/connectivity/pattern-private-end-to-end.md)
- [decision-public-vs-private](../../40-decision-guides/decision-public-vs-private.md)
