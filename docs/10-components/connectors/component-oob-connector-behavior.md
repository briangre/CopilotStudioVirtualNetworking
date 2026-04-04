# Component: Out-of-Box (OOB) Connector Behavior

## Purpose

Describes the behavior, capabilities, and constraints of the pre-built (OOB) Power Platform connector for Databricks or MCP endpoints, as it exists in the connector gallery without customization.

## When to Use

Reference this component when:
- Evaluating whether an OOB connector is sufficient for your scenario.
- Understanding what the OOB connector sends and receives.
- Diagnosing unexpected behavior from the OOB connector.

## Inputs / Prerequisites

- Power Platform environment with internet access (for public path) or a configured VNet data gateway (for private path).
- Valid Databricks workspace URL.
- Authentication configured as described in [component-pp-auth-models](../power-platform/component-pp-auth-models.md).
- Connector enabled by the Power Platform admin in the tenant's Data Loss Prevention (DLP) policy.

## Outputs / What "Done" Looks Like

- Connector action returns a structured response (JSON) from the Databricks MCP endpoint.
- No errors in the Power Platform connector test.
- DLP policy does not block the connection.

## Connector Capabilities

- Invokes Databricks MCP tool-call endpoints (`mcpgenie`, `mcpsql`) using the standard request/response format.
- Handles OAuth token acquisition and refresh automatically.
- TODO: confirm the exact set of actions exposed by the OOB connector (e.g., list tools, call tool, list sessions).

## Configuration Steps

1. In Power Apps / Power Automate / Copilot Studio, search for the Databricks connector in the connector gallery.
2. Select "Add a connection" and provide the Databricks workspace URL and auth credentials.
3. Confirm the connection status is "Connected".
4. Add a connector action to your flow/agent and configure the required inputs.

## Validation Steps

1. Use the built-in "Test" feature in Power Automate to invoke a connector action.
2. Verify the response body matches the expected schema.
3. Check the Power Platform admin center for DLP policy violations.
4. Confirm token refresh works by waiting for the initial token to expire and retrying. TODO: confirm token TTL.

## Known Limitations

- The OOB connector action set is fixed; you cannot add custom actions without building a custom connector.
- Private network scenarios may require a VNet data gateway or OPDG, which is not configured by default.
- DLP policies may block the connector depending on tier classification. TODO: confirm connector DLP tier.
- Rate limits are enforced by Power Platform and may differ from Databricks API limits. TODO: confirm rate limits.

## Related

- [component-custom-connector-auth](component-custom-connector-auth.md)
- [component-pp-auth-models](../power-platform/component-pp-auth-models.md)
- [decision-which-connector](../../40-decision-guides/decision-which-connector.md)
