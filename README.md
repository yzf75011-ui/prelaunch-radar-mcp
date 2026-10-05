# New Shopify Stores — MCP tool for AI agents

Give your agent a live feed of **new Shopify stores, including pre-launch ones** (still behind
their password page), plus a **Shopify checker** for any list of domains. One flat JSON object
per store, pay per result, no personal data.

The tool runs on Apify and is served by the official Apify MCP server, so any MCP client
(Claude Desktop, Claude Code, Cursor, VS Code, Windsurf, n8n, LangChain, CrewAI) can use it.

> Want the full raw database directly? [Download the full snapshot (4,000+ stores) for $4.99](https://slama13.gumroad.com/l/full-snapshot) — one CSV, no subscription, no agent needed.

> **Also in this repo:** [Brand Lookalike Watch](brand-lookalike-watch/) — phishing and
> typosquatting domains imitating a brand, from the same CT logs, as an MCP tool.

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
- "Give me only the stores added since my last call." → `{"mode": "feed", "cursor": "<nextCursor from the previous run's OUTPUT>"}` (each store is billed once; start with `"cursor": "latest"`, which is free)
- "Which of these domains run on Shopify, and when were they registered?" → `{"mode": "enrich", "domains": ["example.com"]}`
- `maxItems` caps billed results, so spend is bounded before the call.

Fields: `store_domain`, `store_name`, `status` (`live` | `pre-launch`), `niche`, `country`,
`region`, `currency`, `products`, `collections`, `ships_to`, `product_types`, `sample_titles`,
`domain_registered` (RDAP), `first_detected`, `status_checked_at` and `status_since` (dated status checks), `cursor`.

## Automate it: new stores into Clay, n8n, Make or your CRM

No server to run. Import [`integrations/n8n/new-shopify-stores-daily.json`](integrations/n8n/new-shopify-stores-daily.json)
into n8n (**Workflows → Import from file**):

1. **Every morning at 8** triggers the workflow.
2. **Set your settings** holds everything you change: `webhookUrl`, `startCursor` (`0` = the last
   30 days, `latest` = only stores added from now on), `status` (`any`, `pre-launch` or `live`),
   `countries` (e.g. `US, GB`, empty = all) and `maxItems` (100), which caps what one run can bill.
3. **Load cursor** reads the position saved by the last successful run.
4. **Get new Shopify stores (Apify)** calls the Actor with that cursor, so it returns and bills only
   the stores added since. Add a *Header Auth* credential:
   name `Authorization`, value `Bearer <your Apify API token>`.
5. **Keep store fields** keeps domain, name, status and its check date, niche, country, currency,
   products, dates and cursor.
6. **Send each store to your webhook** posts one JSON object per store to any URL:
   a Clay table webhook, a Make custom webhook, Slack, HubSpot, your own endpoint.
7. **Save cursor** moves the position forward once the webhook calls succeed. n8n keeps it only in
   active (scheduled) runs, so a manual test run starts from `startCursor` again.

### Lead scoring version: ranked stores into Google Sheets and Slack

[`integrations/n8n/new-shopify-stores-lead-scoring.json`](integrations/n8n/new-shopify-stores-lead-scoring.json)
goes further: it uses the same cursor, so each store arrives and is billed once, scores each new store from 0 to 10
(pre-launch, your target niches and countries, small catalog, ships abroad, young domain) with
the reasons written out, logs every store in Google Sheets (updated by domain), sends one Slack
alert per hot store and one digest for the warm ones. Sample data is pinned, so a first test
run needs no Apify token.

Also in the official n8n template gallery, ready to import in one click:
[n8n.io/workflows/19966](https://n8n.io/workflows/19966). The gallery copy follows n8n's review
schedule and may still use the older `sinceDays` setting; the files in this repository are the latest.

A `niches` filter can be added to the Apify request body. The feed holds
business-level public data only; any contact enrichment you add downstream is your own
processing, under your own compliance.

## Links

- Apify Store page: https://apify.com/prelaunch-radar/new-shopify-stores-pre-launch-radar
- Free sample (frozen on 2026-10-05) + stats on Hugging Face: https://huggingface.co/datasets/jowalker/new-shopify-stores
- Want the full raw database directly? [Download the full snapshot (4,000+ stores) for $4.99](https://slama13.gumroad.com/l/full-snapshot)
- Prefer a spreadsheet? Monthly CSV membership: https://slama13.gumroad.com/l/smjncx

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
