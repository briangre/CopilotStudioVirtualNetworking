# Azure API Management as a Proxy Layer

## Why APIM

Azure API Management (APIM) is the supported mechanism for bridging Copilot Studio (public, Microsoft-managed) and Azure Databricks (potentially private or in a VNet). It solves three problems simultaneously:

1. **Network bridging** — APIM can be VNet-integrated and reach private endpoints
2. **Auth translation** — APIM can validate inbound tokens and inject different credentials toward Databricks
3. **API contract normalization** — APIM can reshape Databricks' async polling APIs into synchronous responses

---

## When You Need APIM

| Scenario | APIM Required? |
|---|---|
| Databricks has a public endpoint | No (optional for auth/rate limiting) |
| Databricks is on a private VNet | ✅ Yes (or equivalent proxy) |
| MCP server is on a private VNet | ✅ Yes |
| You need rate limiting / quota management | Recommended |
| You need centralized logging of agent calls to Databricks | Recommended |
| You need to inject a Databricks service principal token without exposing it to Copilot Studio | ✅ Yes |

---

## APIM Tier Selection

APIM tier determines VNet integration capability:

| Tier | VNet Integration | Suitable for Private Databricks? |
|---|---|---|
| Consumption | ❌ No VNet | No |
| Developer | ✅ VNet injection | Yes (non-production only; no SLA) |
| Basic / Standard | ❌ No VNet | No |
| Standard v2 | ✅ VNet integration (outbound) | Yes |
| Premium | ✅ VNet injection (full) | Yes (production recommended) |

> ⚠️ Standard v2 supports outbound VNet integration (APIM calls out through your VNet) but does not support inbound private endpoint in the same way as Premium. Validate this matches your security requirements.

---

## Architecture Patterns

### Pattern 1: APIM in Front of Private Databricks

```
Copilot Studio
    │
    │  HTTPS POST /databricks/sql/statements
    │  Authorization: Bearer <Entra ID token for APIM>
    ▼
Azure API Management (public endpoint)
    │  [Policy: Validate JWT]
    │  [Policy: Set Authorization header → Databricks PAT or Entra token]
    │  [Policy: Forward to private Databricks URL]
    ▼
Databricks Private Endpoint (via VNet peering / APIM VNet injection)
    │
    ▼
SQL Warehouse / Databricks REST API
```

**What APIM policies handle:**
- Validating the inbound Entra ID token (caller must be authenticated to APIM)
- Stripping the inbound token
- Injecting the Databricks credential (PAT or service principal token)
- Routing to the private Databricks URL

### Pattern 2: APIM in Front of Private MCP Server

```
Copilot Studio
    │
    │  HTTPS POST /mcp/tools/run_sql_query
    │  Authorization: Bearer <Entra ID token>
    ▼
Azure API Management (public endpoint)
    │  [Policy: Validate JWT]
    │  [Policy: Forward headers or inject auth]
    ▼
MCP Server (private VNet)
    │
    ▼
Databricks (private endpoint)
```

This pattern adds latency but is the only supported way to use a private MCP server with Copilot Studio.

---

## Key APIM Policy Patterns

### Validate Inbound Entra ID Token

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401">
  <openid-config url="https://login.microsoftonline.com/{tenant-id}/.well-known/openid-configuration" />
  <audiences>
    <audience>api://{your-apim-app-registration-client-id}</audience>
  </audiences>
  <issuers>
    <issuer>https://sts.windows.net/{tenant-id}/</issuer>
  </issuers>
</validate-jwt>
```

### Inject Databricks Bearer Token (PAT stored in APIM Named Values)

```xml
<set-header name="Authorization" exists-action="override">
  <value>Bearer {{databricks-pat-named-value}}</value>
</set-header>
```

> ⚠️ Storing a PAT in APIM Named Values is acceptable when the PAT is scoped to a service principal with least-privilege permissions and rotated regularly. For higher security, use APIM + Managed Identity to acquire a token at request time. See [06-auth.md](./06-auth.md).

### Acquire Token via Managed Identity (Preferred)

```xml
<authentication-managed-identity resource="2ff814a6-3304-4ab8-85cb-cd0e6f879c1d" />
```

The GUID `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d` is the Azure Databricks first-party application ID in Entra ID. APIM's managed identity must be granted the appropriate Databricks permissions.

---

## Handling the Asynchronous SQL Execution Pattern via APIM

The Databricks SQL Statement Execution API is asynchronous. Callers must poll for results. This is not a natural fit for Copilot Studio's synchronous action model.

APIM can mask this complexity with a **retry/polling policy**:

```xml
<!--
  count:    Maximum number of polling attempts before returning a timeout error.
            Set based on expected query duration: count × interval ≤ acceptable wait time.
            Example: count=15, interval=3 → up to 45 seconds of polling.
  interval: Seconds between each poll. Increase for long-running analytical queries.
            Keep low (1–3s) for interactive lookups; increase (5–10s) for batch queries.
-->
<retry condition="@(context.Response.StatusCode == 200 && (string)context.Response.Body.As<JObject>()["status"]["state"] == "RUNNING")" count="10" interval="2">
  <send-request ...>
    <!-- Poll the statement status endpoint -->
  </send-request>
</retry>
```

This converts the async Databricks pattern into a synchronous response for Copilot Studio. Tune `count` and `interval` to match your expected query latency — short interactive queries (≤10s) can use `count=5, interval=2`; longer analytical queries may need `count=20, interval=5`.

> ⚠️ APIM has a 240-second backend timeout. Queries that take longer than this will return a timeout error. Design long-running queries to run as Databricks jobs instead.

---

## APIM and MCP: Important Constraint

If you use APIM in front of an MCP server, Copilot Studio must be configured to call the **APIM endpoint URL**, not the MCP server URL directly.

The APIM endpoint must:
- Serve the MCP manifest at the path Copilot Studio expects (e.g., `/.well-known/mcp.json`)
- Pass-through or proxy all MCP tool call routes
- Not alter the MCP response format (APIM policies must be non-destructive to MCP JSON)

---

## Operational Considerations

| Concern | Detail |
|---|---|
| Latency | APIM adds 5–30ms overhead per call; acceptable for most agent scenarios |
| Cost | APIM tiers with VNet integration (Standard v2, Premium) are significantly more expensive than Consumption. Factor this into your architecture. |
| Availability | Premium APIM with multi-region or zone redundancy recommended for production |
| Token rotation | If using PAT stored in Named Values, implement a rotation procedure. Prefer Managed Identity. |
| Logging | Enable APIM Application Insights integration to capture all agent-to-Databricks calls for audit and debugging |

---

## When APIM Is Not the Right Answer

APIM adds cost and complexity. Skip it if:

- Your Databricks workspace has a public endpoint and your security team permits allowlisting Power Platform IPs
- You are building a proof of concept or prototype
- Your latency requirements cannot tolerate the APIM hop

In these cases, call Databricks directly from the Custom Connector or HTTP action.
