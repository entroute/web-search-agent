---
name: web-search-agent
description: Search the live web via a verified x402 pay-per-request endpoint. Use when the user asks the agent to find recent information online, look up current events, research a topic, or fetch search results — without an API key for Exa, Serper, Brave, or Bing.
---

# Web search via EntRoute

When the user needs live web results, use this skill to query a ranked, fulfillment-verified x402 search endpoint.

## Steps

1. **Extract the search query** from the user's request. Keep it concise and targeted — 3–10 words usually works best. Note any constraints (recency, number of results, site-restricted).

2. **Discover the best endpoint.** Call EntRoute's `/discover`:

   ```bash
   curl -s -X POST https://api.entroute.com/discover \
     -H "Content-Type: application/json" \
     -d '{
       "capability_id": "web.search",
       "constraints": {"network": "base", "verified_only": true},
       "preferences": {"ranking_preset": "balanced"}
     }'
   ```

   Take `ranked_endpoints[0]`. The response includes `url`, `method`, `payment`, and a `sample_request` showing exactly which params the endpoint expects (commonly `query`, sometimes `q`, plus optional `num_results`, `num_pages`, or `mode`).

3. **Build the request matching `sample_request`.** Parameter shapes differ across providers — always use the shown schema rather than guessing.

4. **Pay with x402.** With `@entroute/mcp-server` or `@entroute/sdk-agent-ts` installed, `call_paid_api` / `discoverAndCall` handles payment. Otherwise, the user needs a funded Base wallet + x402 client.

5. **Parse and return.** Different providers return different shapes. Use `sample_response` from discovery to map fields. Typical structure: `{ results: [{ title, url, snippet, ... }] }`.

## Preferred approach

With Claude Code + `@entroute/mcp-server`, use the MCP tools `discover_paid_api` and `call_paid_api` directly. They handle discovery + payment in one step.

## Ranking preset tip

- For research/analysis: `preset: balanced` (the default).
- For snappy lookups in a conversational agent: `preset: speed`.
- For important factual queries where accuracy > latency: `preset: reliability`.

## Notes

- Default network is Base; pricing is USDC. Search endpoints typically cost $0.002–$0.01 per query.
- If results feel shallow, re-run discovery — rankings refresh every 10 minutes and a different provider may be top.
- Full docs: https://entroute.com/docs/quickstart
