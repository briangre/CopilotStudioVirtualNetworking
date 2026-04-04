# Decision Guide: Which Connector Type Should I Use?

Use this guide to choose between an **Out-of-Box (OOB)** connector and a **Custom connector**.

## Decision Tree

```
Start
  │
  ▼
Is there an OOB connector in the Power Platform gallery
that supports the Databricks MCP endpoint you need?
  │
  ├─ YES ──► Does the OOB connector support the auth scheme
  │          required by your Databricks workspace?
  │            │
  │            ├─ YES ──► Does the OOB connector work with
  │            │          your network path (public/private)?
  │            │            │
  │            │            ├─ YES ──► Use OOB connector ✅
  │            │            │         See component-oob-connector-behavior
  │            │            │
  │            │            └─ NO  ──► Use Custom connector ⚙️
  │            │                       See component-custom-connector-auth
  │            │
  │            └─ NO  ──► Use Custom connector ⚙️
  │
  └─ NO  ──► Use Custom connector ⚙️
```

## OOB Connector

**Use when:**
- A supported OOB connector exists for your Databricks MCP endpoint.
- The auth model is OAuth (client credentials or OBO) and Power Platform can reach the Databricks endpoint over the public network.
- You want minimal setup and maintenance.

**Limitations:**
- Limited ability to customize request/response handling.
- May not support private network paths without additional infrastructure. See [component-public-vs-private](../10-components/networking/component-public-vs-private.md).
- Auth configuration is fixed to the connector's supported schemes.

→ [component-oob-connector-behavior](../10-components/connectors/component-oob-connector-behavior.md)

## Custom Connector

**Use when:**
- No suitable OOB connector exists.
- You need to route traffic through APIM or a private endpoint.
- You require a non-standard auth configuration.
- You need to transform request/response payloads.

**Limitations:**
- Requires developer effort to build and maintain the OpenAPI spec.
- Must be registered and shared within the Power Platform environment.
- Certificate management and secret rotation fall on the connector owner.

→ [component-custom-connector-auth](../10-components/connectors/component-custom-connector-auth.md)

## Related

- [decision-public-vs-private](decision-public-vs-private.md)
- [matrix-configurations](../50-matrix/matrix-configurations.md)
