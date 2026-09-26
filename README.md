# New Shopify Stores — MCP tool for AI agents

Give your agent a live feed of **new Shopify stores, including pre-launch ones** (still behind
their password page), plus a **Shopify checker** for any list of domains. One flat JSON object
per store, pay per result, no personal data.

The tool runs on Apify and is served by the official Apify MCP server, so any MCP client
(Claude Desktop, Claude Code, Cursor, VS Code, Windsurf, n8n, LangChain, CrewAI) can use it.

## Connect in one paste

**Remote (recommended, sign in with Apify in the browser):** add this to your client's MCP config

```json
{
  "mcpServers": {
    "new-shopify-stores": {
      "url": "https://mcp.apify.com/?tools=prelaunch-radar/new-shopify-stores-pre-launch-radar"
    }
  }
}
```

**Local (stdio, with an Apify API token):** see [`clients/local-stdio.json`](clients/local-stdio.json).

Claude Code: `claude mcp add --transport http new-shopify-stores "https://mcp.apify.com/?tools=prelaunch-radar/new-shopify-stores-pre-launch-radar"`

## What the agent can ask

- "List today's new Shopify stores in the US that are still pre-launch." → `{"mode": "feed", "sinceDays": 1, "status": "pre-launch", "countries": ["US"]}`
- "Which of these domains run on Shopify, and when were they registered?" → `{"mode": "enrich", "domains": ["example.com"]}`
- `maxItems` caps billed results, so spend is bounded before the call.

Fields: `store_domain`, `store_name`, `status` (`live` | `pre-launch`), `niche`, `country`,
`region`, `currency`, `products`, `collections`, `ships_to`, `product_types`, `sample_titles`,
`domain_registered` (RDAP), `first_detected`.

## Automate it: new stores into Clay, n8n, Make or your CRM

No server to run. Import [`integrations/n8n/new-shopify-stores-daily.json`](integrations/n8n/new-shopify-stores-daily.json)
into n8n (**Workflows → Import from file**):

1. **Every morning** triggers the workflow at 08:00.
2. **Get new Shopify stores** calls the Actor with `{"mode": "feed", "sinceDays": 1, "maxItems": 100}`.
   Add a *Header Auth* credential: name `Authorization`, value `Bearer <your Apify API token>`.
   `maxItems` caps what one run can bill.
3. **Send each store to your webhook** posts one JSON object per store to any URL:
   a Clay table webhook, a Make custom webhook, Slack, HubSpot, your own endpoint.

Filters (`status`, `countries`, `niches`) go in the same JSON body. The feed holds
business-level public data only; any contact enrichment you add downstream is your own
processing, under your own compliance.

## Links

- Apify Store page: https://apify.com/prelaunch-radar/new-shopify-stores-pre-launch-radar
- Prefer a spreadsheet? Always-fresh CSV membership: https://slama13.gumroad.com/l/smjncx

## How it works

Public Certificate Transparency logs (RFC 6962) → public DNS records pointing to Shopify →
official RDAP registry → each store's public `/meta.json` and `/products.json`.

## LEGAL DISCLAIMER

This dataset is provided for market research, trend analysis, and competitive intelligence
purposes only, on an "as-is" and "as-available" basis without warranties of any kind.
Records originate exclusively from publicly accessible sources (Certificate Transparency
logs, public DNS and RDAP registries, and stores' public storefront metadata); no personally
identifiable information (PII) is collected or sold.
Independently developed, not affiliated with, authorized, or endorsed by Shopify Inc. or
Apify. By accessing this data, you agree that the provider shall not be liable for any direct
or indirect damages resulting from its use; in any event, liability is limited to the purchase
price.
