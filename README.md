# Offers MCP — by Dexter Works

**Am I getting the best deal on this exact product?**

Offers is an MCP server that gives AI assistants normalized purchase offers
for a specific product. Give it a product — free text like
`Sony WH-1000XM6 black`, or a GTIN/UPC/EAN — plus an optional US zip code, and
it returns structured offers across merchants:

- **Price, shipping, and landed cost** (the number that actually matters)
- **Delivery window for your zip code**
- **Merchant trust tier** (curated A/B/C — no pay-to-rank)
- **Match confidence** per offer, so your agent knows when to trust a result
- A computed **best trusted offer** and **fastest trusted offer**
- Honest freshness: every offer carries `checked_at` and cache status

No signup. No API key. No commitment.

## Quick start

The hosted server speaks streamable HTTP at:

```
https://offers-engine.dexter-works.com/mcp
```

**Claude Code**

```bash
claude mcp add --transport http offers https://offers-engine.dexter-works.com/mcp
```

**Claude Desktop / other MCP clients** — add a remote server with the URL
above, or via [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) for
clients that only support stdio:

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

Then ask your assistant something like:

> Am I getting a good deal on the Sony WH-1000XM6 at $398? I'm in 83702.

## REST API

The same engine is available over plain HTTP:

```bash
curl "https://offers-engine.dexter-works.com/api/search_offers?q=Sony%20WH-1000XM6%20black&zip=83702"
```

## Tools

| Tool | Input | Output |
|---|---|---|
| `search_offers` | `query` (text or GTIN), `zip` (optional, 5-digit US) | Normalized offers + best/fastest trusted offer |

More tools (`identify_product` — GTIN/URL/image resolution and variant
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

- **Sources:** offers currently come from Google Shopping, with eBay and
  Best Buy integrations rolling out (each response lists exactly what was
  queried in `meta.sources_queried`). GTIN inputs are resolved via UPCitemdb.
  More sources are added as the normalization layer proves out.
- **Latency:** a cold query takes roughly 5–15 seconds (live upstream
  lookups); repeated queries hit a short-lived cache and return in
  milliseconds. Configure generous tool timeouts.
- **Rate limits / fair use:** free during v0 with no hard published limit —
  please stay under ~50 requests/day per user. Heavy or commercial use:
  contact us first. The endpoint may evolve; breaking changes will be
  announced in this repo.
- **REST and MCP return the same JSON shape** (the MCP tool wraps it as text
  content).
- **Note when curling `/mcp` directly:** responses are SSE-framed
  (`event: message` / `data: {...}`) per MCP streamable HTTP; MCP clients
  handle this for you.

## Status & support

Early v0 — US consumer goods, a small set of sources done honestly. If a
field can't be trusted, it's `null` rather than guessed.

Found a bad result, wrong price, or a bug? **Open a
[GitHub issue](https://github.com/dexterworks/offers-mcp/issues)** — bad-match
reports with the exact query are especially valuable right now.

Built by [Dexter Works](https://dexter-works.com).
