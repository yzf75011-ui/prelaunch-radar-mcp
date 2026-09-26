# Brand Lookalike Watch — MCP tool for AI security agents

Give your agent the **newly certified domains that imitate a brand**: typosquatting
(`paypall.com`), homoglyphs and IDN (`paypa1.com`, Cyrillic look-alikes), combosquatting
(`paypal-login-secure.com`) and TLD variants (`nike.shop`). The domains come from public
**Certificate Transparency logs**, so they really exist: no guessed permutations. Each result
carries a 0–100 risk score with readable signals, public DNS status and the RDAP registration
date. Pay per result, no personal data.

The tool runs on Apify ([store page](https://apify.com/prelaunch-radar/brand-lookalike-watch))
and is served by the official Apify MCP server.

## Connect in one paste

```json
{
  "mcpServers": {
    "brand-lookalike-watch": {
      "url": "https://mcp.apify.com/?tools=prelaunch-radar/brand-lookalike-watch"
    }
  }
}
```

Claude Code: `claude mcp add --transport http brand-lookalike-watch "https://mcp.apify.com/?tools=prelaunch-radar/brand-lookalike-watch"`

## What the agent can ask

- "Any new phishing domains imitating our brand since yesterday?" → `{"brands": ["acmebank"], "ownDomains": ["acmebank.com"], "sinceDays": 1, "minScore": 40}`
- "Score these suspicious domains against our brand." → `{"brands": ["acmebank"], "mode": "check", "domains": ["acmebank-login.xyz"]}`
- `maxItems` caps billed results, so spend is bounded before the call.

Fields: `domain`, `unicode`, `brand`, `match_type` (`typosquat` | `homoglyph` | `combosquat` |
`tld-variant`), `matched_on`, `registrable_domain`, `risk_score`, `risk_signals`, `resolves`,
`ip`, `domain_registered`, `domain_age_days`, `certificate_seen`.

## Coverage

Let's Encrypt certificates sampled from public CT logs every 2 hours, kept 7 days. An
early-warning feed, not an exhaustive registry. Scores are signals, not verdicts.

## Legal

Independent tool, not affiliated with any brand users monitor. Public infrastructure data only
(CT logs, DNS, RDAP), no personal data. Provided "as is", without warranty; liability limited to
the amount paid for the run.
