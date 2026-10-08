---
name: omniroute
description: Use when web search is needed but built-in search tools are unavailable, rate-limited, or a specific search provider is wanted, when an OmniRoute gateway's model catalog is needed, for an ad-hoc one-shot LLM call as a sub-step in a larger task (never instead of the session model), or when the OmniRoute gateway is down or returning 502 and needs a health check.
---

# OmniRoute gateway

REST access to an OmniRoute LLM gateway: web search with sources, the model catalog, and one-shot LLM calls.

## Config

```bash
BASE="${OMNIROUTE_BASE_URL:-http://127.0.0.1:20128}"
if [ -n "$OMNIROUTE_API_KEY" ]; then K="$OMNIROUTE_API_KEY"; else K=$(cat ~/.omniroute/agent-skills.key); fi   # never print or echo the key
```

If your tool shell doesn't persist state between calls, re-inline both variables per command.

## When to use / not use

Use for: web search when built-in tools fail or a specific provider is needed; listing what models the gateway serves; one-off LLM calls (classify, summarize, extract) inside a task.

Do NOT use for: the session's own model routing (the CLI's native provider config already does that) — and note that the gateway's memory and skill-execution surfaces are MCP-only, not reachable over REST.

## Web search (returns sources)

```bash
curl -s -m 45 -X POST "$BASE/v1/search" \
  -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d '{"query":"<what to research>"}'
```

Response shape: `{"provider":"…","query":"…","results":[{"title","url","display_url","snippet"}]}`. Snippets can run 30+ lines each — extract bounded: `| jq -r '.results[0:3][] | "\(.title) — \(.url)"'`, and read a specific snippet only when needed: `| jq -r '.results[0].snippet'`. Cite the `url` values you used.

The provider is auto-selected when omitted (free on default setups). To force one, pass `"provider":"<value>"` — valid values depend on the search providers configured on your instance, and an unknown value fails fast with `{"error":{"message":"Unknown search provider: …","type":"invalid_request_error"}}` so you can retry. Slugs verified on a reference instance (3.8.51): `serper`, `anysearch`, `zai`, `brave`, `exa` (exa can take 30s+). There is no result-count parameter (`limit` is ignored).

## Model catalog

```bash
curl -s -m 10 "$BASE/v1/models" -H "Authorization: Bearer $K" | jq -r '.data[].id' | grep -iE '<pattern>'
```

The catalog can hold 1000+ ids — always filter with `grep`; never dump the raw list into context. Combo ids like `auto/best-coding` route to a picked backend automatically; plain ids are direct models.

## One-shot chat call

```bash
curl -s -m 60 -X POST "$BASE/v1/chat/completions" \
  -H "Authorization: Bearer $K" -H "Content-Type: application/json" \
  -d '{"model":"auto/best-coding","messages":[{"role":"user","content":"<prompt>"}]}'
```

Standard OpenAI shape — content at `.choices[0].message.content`.

## Troubleshooting

- Health (no auth needed): `curl -s -m 5 "$BASE/api/health"` → `{"status":"ok",…}`.
- Connection refused or 502: check health, restart your OmniRoute service (e.g. `launchctl kickstart -k gui/$(id -u)/im.omniroute.server` when installed as that LaunchAgent), then re-check health.
- 401 (or the keyfile is absent): recreate with `omniroute api api-keys post-api-keys --body '{"name":"agent-skills","scopes":["execute:search"]}'`, write the returned `key` to `~/.omniroute/agent-skills.key` (chmod 600). Never print it.
