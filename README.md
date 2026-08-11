# Offers MCP

**Every offer. One call.**

Offers MCP gives deep purchase intelligence to AI agents, applications, and humans. Point
it at a product (free text like `Sony WH-1000XM6 black`, or a GTIN/UPC/EAN
barcode), add an optional US zip code, and one call scans merchants in
parallel to return live prices, true cost, delivery options, and trust
insights. Robust, private, efficient results for agents, applications, and humans to make
informed purchase decisions.

**Robust. Built to be trusted:**

- **Live prices with true landed cost** (price + shipping, the number that actually matters)
- **Delivery window for your zip code**
- **Curated merchant trust tier** (A/B/C, no pay-to-rank), with untrusted listings filtered
- **Confidence scores** on the product match and on every offer, so your agent knows when to assert and when to hedge. When we're not sure, the response says so instead of guessing
- A computed **best trusted offer** and **fastest trusted offer**, plus the full option list. The final call stays with your user
- **Provenance on everything:** every offer names its source and the moment it was checked (`checked_at`, cache status)

**Private. Nobody watches you shop:** no per-user tracking, no raw IPs
stored; searches stay private: never sold, shared, or used to retarget you
(details in [Privacy](#privacy)).

**Efficient. Shop thousands of stores in seconds:** one input fans out to
multiple sources at once and comes back normalized into one structured
contract you can build against. Repeat lookups return instantly. Use it for a
cross-vendor price check from the store aisle or programmatic lookups inside
your own application. Same tool call, same answers.

Free during v0. No signup. No API key. No commitment.

## Quick start

The hosted server speaks streamable HTTP at:

```
https://offers-engine.dexter-works.com/mcp
```

### Claude Code

```bash
claude mcp add --transport http offers https://offers-engine.dexter-works.com/mcp
```

### Claude Desktop / claude.ai (no terminal needed)

1. Open **Settings** → **Connectors** (paid plans).
2. Click **Add custom connector**.
3. Name it `Offers`, paste the URL `https://offers-engine.dexter-works.com/mcp`, and save.
4. In a new chat, make sure Offers is enabled in the tools/connectors menu, then just ask your shopping question.

### ChatGPT

1. **Settings** → **Apps & Connectors** → **Advanced settings** → enable **Developer mode** (Plus/Pro/Team plans).
2. Back in **Connectors**, choose **Create** / add a custom connector.
3. Name it `Offers`, MCP server URL `https://offers-engine.dexter-works.com/mcp`, authentication: **none**.
4. In a chat, enable the Offers connector from the tools menu and ask away.

### Cursor

Add to `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global):

```json
{ "mcpServers": { "offers": { "url": "https://offers-engine.dexter-works.com/mcp" } } }
```

### VS Code (GitHub Copilot)

Add to `.vscode/mcp.json`:

```json
{ "servers": { "offers": { "type": "http", "url": "https://offers-engine.dexter-works.com/mcp" } } }
```

### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{ "mcpServers": { "offers": { "serverUrl": "https://offers-engine.dexter-works.com/mcp" } } }
```

### Any stdio-only MCP client

Use [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) as a bridge:

```json
{
  "mcpServers": {
    "offers": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://offers-engine.dexter-works.com/mcp"]
    }
  }
}
```

> Client UIs move fast. If a menu path above is stale,
> [open an issue](https://github.com/dexterworks/offers-mcp/issues) and we'll fix it.

Then ask your assistant something like:

> Use Offers MCP to check if I'm getting a good deal on the Sony WH-1000XM6 at $398. I'm in 83702.

Naming the tool matters: assistants have their own web search and will often
reach for it first on shopping questions. Saying "use Offers MCP" (or enabling
only this connector for the chat) makes sure your question hits live
structured offers instead of search snippets.

## REST API

The same engine is available over plain HTTP:

```bash
curl "https://offers-engine.dexter-works.com/api/search_offers?q=Sony%20WH-1000XM6%20black&zip=83702"
```

## Tools

| Tool | Input | Output |
|---|---|---|
| `search_offers` | `query` (text or GTIN), `zip` (optional, 5-digit US) | Normalized offers + best/fastest trusted offer |

More tools (`identify_product` for GTIN/URL/image resolution and variant
families) are on the roadmap.

## Response shape (abridged)

```json
{
  "product": { "name": "Sony WH-1000XM6…", "brand": "Sony", "resolution_confidence": 0.9 },
  "offers": [
    {
      "merchant": { "name": "Best Buy", "trust": { "tier": "A", "score": 0.95 } },
      "price": { "amount": 398.0, "list_price": 449.99, "discount_pct": 11.6 },
      "fulfillment": {
        "landed_cost": 398.0,
        "delivery": { "zip": "83702", "min_days": 2, "max_days": 2, "basis": "merchant_stated" }
      },
      "variant_match": { "confidence": 0.98 },
      "provenance": { "checked_at": "2026-08-11T19:42:03Z", "cache": "miss" }
    }
  ],
  "summary": {
    "best_offer_id": "dw:off_…",
    "recommendation": "Best Buy at $398.00 with free shipping is the best trusted deal."
  }
}
```

## Sources, limits, and expectations

- **Sources (quality data, not screen-scraping):** we work through official
  merchant and catalog partner programs and structured data sources so
  results stay accurate rather than brittle. Today offers come from Google
  Shopping, with retailer-direct integrations (Best Buy, Walmart) rolling
  out; GTIN inputs are resolved via Icecat and UPCitemdb. Each response
  lists exactly what was queried in `meta.sources_queried`, and where
  sources disagree or fall short, the response says so.
- **Latency:** a cold query takes roughly 5–15 seconds (live upstream
  lookups); repeated queries hit a short-lived cache and return in
  milliseconds. Configure generous tool timeouts.
- **Rate limits / fair use:** free during v0 with no hard published limit;
  please stay under ~50 requests/day per user. Heavy or commercial use:
  contact us first. The endpoint may evolve; breaking changes will be
  announced in this repo.
- **REST and MCP return the same JSON shape** (the MCP tool wraps it as text
  content).
- **Note when curling `/mcp` directly:** responses are SSE-framed
  (`event: message` / `data: {...}`) per MCP streamable HTTP; MCP clients
  handle this for you.

## Privacy

We built Offers for ourselves first, and we treat your searches the way we'd
want ours treated.

- **What we see:** the product you searched, the zip code you optionally
  provide, and coarse technical context (which kind of client connected,
  country/region). We don't track individuals or build per-user shopping
  profiles, and anonymous use requires no account at all.
- **What we don't keep:** raw IP addresses are never stored; unique usage is
  counted with a hash that changes daily and can't be reversed.
- **What it's used for:** one thing: making results better (match accuracy,
  merchant coverage, speed).
- **What we will never do:** sell your data, share it with advertisers or
  data brokers, or use your searches to retarget you. Full stop.
- **Merchants can't watch you compare.** All price comparison happens on our
  servers; a merchant learns about you only if you click through to their
  site to buy. Some outbound links may earn us an affiliate commission at no
  cost to you, and commissions never influence ranking (trust tiers and landed
  cost decide, and that logic treats every merchant the same).

Questions or want something deleted? [hello@dexter-works.com](mailto:hello@dexter-works.com).

## Status & support

Early v0: US consumer goods, a small set of sources done honestly. If a
field can't be trusted, it's `null` rather than guessed.

Found a bad result, wrong price, or a bug? **Open a
[GitHub issue](https://github.com/dexterworks/offers-mcp/issues)**. Bad-match
reports with the exact query are especially valuable right now.

Built by [Dexter Works](https://dexter-works.com).
