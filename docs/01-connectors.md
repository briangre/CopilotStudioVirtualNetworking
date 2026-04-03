# Out-of-Box Connectors for Databricks

## Summary

Power Platform ships a built-in Azure Databricks connector. It works for basic use cases but has significant limitations that disqualify it for most production agent scenarios.

---

## The Azure Databricks Connector

**Status: ✅ Supported (with limitations)**

The [Azure Databricks connector](https://learn.microsoft.com/en-us/connectors/azuredatabricks/) is a certified Power Platform connector. It is available in:

- Power Automate (cloud flows)
- Power Apps
- Copilot Studio (via Power Automate action invocation)

### What it does

- Runs notebooks
- Lists clusters, jobs, and notebooks
- Submits and monitors job runs

### What it does not do

| Capability | Supported? |
|---|---|
| Execute SQL queries against a SQL Warehouse | ❌ |
| Call the Genie conversational API | ❌ |
| Call MCP-compatible endpoints | ❌ |
| Work with private VNet-only workspaces | ❌ |
| Return query result sets to a Copilot Studio agent | ❌ (notebook output only) |

### Critical limitation: No SQL execution

The out-of-box connector does not expose the [Databricks SQL Statement Execution API](https://docs.databricks.com/api/workspace/statementexecution). If your agent needs to run SQL against a warehouse, you must use a custom connector or HTTP action.

### Critical limitation: Public endpoint required

The connector authenticates using either:
- API token (basic)
- Azure AD (OAuth 2.0)

However, it sends requests directly to the Databricks workspace URL. If your workspace is on a private VNet with no public endpoint, this connector will not reach it. There is no mechanism in the out-of-box connector to route through APIM or a private link.

---

## Using the Connector in Copilot Studio

Copilot Studio does not invoke Power Platform connectors directly. The supported pattern is:

```
Copilot Studio Agent
    └── Calls a Power Automate Flow (as an action)
            └── Power Automate Flow uses the Azure Databricks connector
```

### Implications

- Latency is added by the Power Automate hop
- Token management is handled by the Power Automate connection (not the agent)
- Return values must be serialized and passed back through the flow return schema
- The agent has no visibility into flow execution errors unless you explicitly surface them

---

## When to Use the Out-of-Box Connector

Use it when:

- You need to trigger a **notebook or job run** (not query results)
- Your workspace has a **public endpoint**
- You are comfortable with **Power Automate as the orchestration layer**
- You do not need real-time SQL results or conversational Databricks responses

---

## When Not to Use It

Do not use the out-of-box connector when:

- You need to execute SQL and return tabular results → Use a **Custom Connector** targeting the SQL Statement Execution API (see [02-endpoints.md](./02-endpoints.md))
- Your workspace is private → Use **APIM as a proxy** (see [05-apim.md](./05-apim.md))
- You want natural language over data → Consider **Genie API** or **MCP** (see [02-endpoints.md](./02-endpoints.md))

---

## Custom Connectors as an Alternative

Power Platform custom connectors allow you to define any REST API surface. For Databricks, you can create a custom connector targeting:

| Target | API | When |
|---|---|---|
| SQL Warehouse | `/api/2.0/sql/statements` | Run SQL, get results |
| Genie | `/api/2.0/genie/spaces/{id}/start-conversation` | Natural language queries |
| Jobs / Workflows | `/api/2.1/jobs/run-now` | Trigger job pipelines |
| Secrets / Config | Databricks REST API | Dynamic config lookup |

Custom connectors support OAuth 2.0 with Entra ID, which is the recommended authentication approach. See [06-auth.md](./06-auth.md) for the full OAuth configuration.
