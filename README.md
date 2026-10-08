# omniroute-skill

An [agent skill](https://github.com/anthropics/skills) for the [OmniRoute](https://github.com/diegosouzapw/OmniRoute) LLM gateway: web search with sources, the model catalog, and one-shot LLM calls — over plain REST, from any coding agent (ZCode, Claude Code, Codex, Cursor, OpenCode, pi, ...).

## Why a skill instead of the MCP server?

OmniRoute ships an MCP server exposing ~110 gateway tools over stdio/HTTP. Mounting it in your agent costs a ~110-tool schema block in every session's context — and most of those tools (gateway admin, compression, Obsidian/Notion, pools, diagnostics) are not what you want mid-task. Mounting it is also all-or-nothing: the tool manifest is not pruned per API key, so a narrowly scoped key still sees all ~110 tool definitions.

This skill gets you the three surfaces an agent actually uses — search, models, one-shot chat — for the price of one skill-description line, with a key scoped to `execute:search`.

## Install

```bash
git clone https://github.com/hongkongkiwi/omniroute-skill.git
mkdir -p ~/.agents/skills
cp -r omniroute-skill/skills/omniroute ~/.agents/skills/
```

`~/.agents/skills/` is the cross-CLI location (ZCode, Codex, pi); Claude Code users can use `~/.claude/skills/` instead. The `skills/<name>/SKILL.md` layout is compatible with skills.sh-style skill registries.

## Setup

1. A running OmniRoute instance (default `http://127.0.0.1:20128` — override with `OMNIROUTE_BASE_URL`).
2. Create a dedicated API key scoped to search:

   ```bash
   omniroute api api-keys post-api-keys --body '{"name":"agent-skills","scopes":["execute:search"]}'
   ```

3. Store the returned key at `~/.omniroute/agent-skills.key` (chmod 600), or export `OMNIROUTE_API_KEY` instead.

## Verified against

OmniRoute 3.8.51 REST API (October 2026): `/v1/search`, `/v1/models`, `/v1/chat/completions`, `/api/health`, and the key-creation CLI shown above were all exercised live. Response shapes are standard OpenAI-style JSON; later versions may drift.

## License

[MIT](LICENSE)
