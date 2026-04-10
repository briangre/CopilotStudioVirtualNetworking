# Component: Public vs. Private Network

## Purpose

Describes the two network path options for Power Platform ↔ Databricks traffic, their infrastructure requirements, and how to choose between them. It is vitally important to understand that there are multiple configuration options for private networking (private endpoint vs vNet Injection). It is up to the reader to determine the appropriate path for their scenario. 

There are also multiple configuration options for Azure Databricks workspaces (serverless vs hybrid). For purposes of this documentation, the simplest path has been chosen. If the desired configuration has dedicated compute resources, then those resources may need their own private networking configuration. This is **not** being covered in this guide.  

## When to Use

Reference this component when:
- Designing the network architecture for a new Power Platform ↔ Databricks integration.
- Determining whether Private Endpoints and Power Platform Virtual Networking are required.
- Troubleshooting network-level connectivity failures.

## Inputs / Prerequisites

### Public Path
- Databricks workspace with public network access enabled.
- Power Platform outbound IP ranges documented and, if needed, added to Databricks network allow-list. [https://learn.microsoft.com/en-us/power-platform/admin/online-requirements]

### Private Path
- Azure VNet in the same region as the Databricks workspace.
- Private Endpoint for the Databricks workspace deployed into a subnet of that VNet.
- Private DNS zone configured for `*.azuredatabricks.net` resolving to the Private Endpoint IP.
- Power Platform virtual netwoks
  - Appropriate paired Azure regions [https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#supported-regions]
  - Each virtual network peered with the Azure Databricks virtual network
  - Each virtual network contains at least one subnet that is delegated to Microsoft.PowerPlatform/enterprisePolicies
  - Each virtual network has a virtual link to the private DNS zone for databricks
 **NOTE:** The Azure networking configuration will be very specific to your enterprise architecture. The above is just a high level discussion about the parts needed to stitch it all together. 

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
Power Platform ──[PP VNet]──[ADB VNet]──[Private Endpoint]──► Databricks
```

## Configuration Steps

**NOTE:** The following configuration steps are intended for a demo environment with no enterprise level networking architecture defined. This is only for testing and non-production scenarios. 

### Public Path

1. Confirm Databricks workspace "Public network access" is `Enabled`.
2. Optionally add Power Platform IP ranges to the Databricks IP access list.
3. Configure the connector with the public workspace URL.

### Private Path

#### Databricks 
1. Create a virtual network (or use an existing one in the same region as Databricks).
2. Deploy a Private Endpoint
    - target sub-resource: databricks_ui_api
    - within subnet for above virtual network
3. Configure a Private DNS zone `privatelink.azuredatabricks.net` and link it to the virtual network.
4. Disable Public Access in the Azure Databricks networking settings and be sure the above private endpoint is assigned in the 'Private endpoint connections'

#### Power Platform
These steps are high-level, for full details, please refer to the official documentation: [https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-setup-configure?tabs=existing%2Csingle&pivots=powershell]
1. Create two virtual networks -- one in each of the required paired regions, for instance EastUS / WestUS.
2. Each virtual network should contain at least one subnet that is delegated to Microsoft.PowerPlatform/enterprisePolicies.
3. Create an Enteprise Policy using these virtual networks as the configuration
4. Bind the desired Power Platform environments to the Enterprise Policy
5. Configure the connector to the desired MCP endpoint.
   
## Validation Steps

1. From a VM inside the Power Platform virtual network, run `nslookup <workspace>.azuredatabricks.net` — expect a private IP.
2. From the same VM, `curl https://<workspace>.azuredatabricks.net/api/2.0/clusters/list` — expect HTTP 200 or 401 (not a network error).
3. Use the Power Platform Enterprise powershell tools for testing DNS and Network connectivity: [https://learn.microsoft.com/en-us/troubleshoot/power-platform/administration/virtual-network]
4. Test the Power Platform connector from Copilot Studio Agent Tool.

## Known Limitations

- Virtual Network and private networking for Power Platform requires a managed environment.
- Private Endpoint DNS resolution can take several minutes to propagate.
- When public access is disabled on Databricks, all management tools (Azure portal, Terraform) must also use the private path.

## Related

- [component-pp-networking](../power-platform/component-pp-networking.md)
- [component-apim-private-ingress-egress](../apim/component-apim-private-ingress-egress.md)
- [pattern-public-direct](../../20-patterns/connectivity/pattern-public-direct.md)
- [pattern-private-end-to-end](../../20-patterns/connectivity/pattern-private-end-to-end.md)
- [decision-public-vs-private](../../40-decision-guides/decision-public-vs-private.md)
