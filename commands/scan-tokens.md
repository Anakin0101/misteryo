---
description: Audit MCP servers and plugins for session-start token overhead. Shows what's loaded at session start and flags high-cost, rarely-used servers. Estimates are approximate — real numbers require probing.
---

# /scan-tokens — MCP & Plugin Token Audit

Show approximately where session-start tokens are going, so the user can disable or proxy heavy MCP servers.

## ⚠️ Honesty up front

Token counts in this audit are **estimates**, not measurements, unless you successfully probe the actual server schemas. Always label numbers clearly as either `ESTIMATED` or `MEASURED`. Never present a guessed number as a measured one.

For true lazy-loading and real token reduction, recommend [contextos](https://github.com/Anzormumladze/contextos) by Anzor Mumladze — it proxies MCP servers and reports actual schema sizes.

## Step 1 — Find MCP configs

Check these paths in order, first one that exists wins per scope:
- Project-level: `.mcp.json` in repo root
- User-level: `~/.claude.json`, `~/.config/claude/claude.json`
- Desktop app (macOS): `~/Library/Application Support/Claude/claude_desktop_config.json`
- Desktop app (Windows): `%APPDATA%\Claude\claude_desktop_config.json`

Parse each config's `mcpServers` block. List each server with its command, args, and source config.

If no configs found, say so and stop.

## Step 2 — Probe where possible (best effort)

For each MCP server, try to get the real tool count:

1. If the server is already running in this Claude Code session, you already see its tools — count them and estimate tokens (roughly 150-300 tokens per tool schema depending on complexity).

2. If the server is not running, **do NOT spawn it just to probe** (side effects, auth prompts, slow). Instead, fall back to the estimate table below and mark the result `ESTIMATED`.

### Fallback estimate table

These are rough ranges based on public MCP servers. Real numbers vary.

| Server | Typical tool count | Estimated tokens |
|--------|-------------------|------------------|
| github | 20–40 | 15k–25k |
| gitlab | 15–30 | 10k–20k |
| figma-remote-mcp | 20–30 | 10k–18k |
| gmail | 10–20 | 5k–12k |
| google-calendar | 8–15 | 4k–10k |
| linear | 15–25 | 8k–15k |
| jira / atlassian | 20–35 | 12k–22k |
| notion | 10–15 | 6k–10k |
| slack | 15–25 | 8k–15k |
| postgres / mysql | 5–10 | 2k–5k |
| filesystem | 8–12 | 3k–6k |
| unknown / custom | ? | mark as "unknown — probe manually" |

## Step 3 — Check plugin overhead

List installed Claude Code plugins and their command/agent counts. Each slash command adds ~100-200 tokens to the command registry at session start; agents are lazy-loaded (near-zero cost until invoked).

## Step 4 — Report

Use this exact format so it is clear what is measured vs estimated:

```
### Session-Start Token Audit

#### MCP Servers
| Server | Tools | Tokens | Source | Status |
|--------|-------|--------|--------|--------|
| github | 30 | ~20k ESTIMATED | ~/.claude.json | ACTIVE |
| figma-remote-mcp | 25 (MEASURED) | ~14k MEASURED | .mcp.json | ACTIVE |
| gmail | 15 | ~8k ESTIMATED | ~/.claude.json | ACTIVE |

Total estimated MCP overhead: ~42k tokens per session start.
Numbers labeled MEASURED come from actually inspecting the running server's tool list. ESTIMATED numbers come from the reference table and may be off by 30-50%.

#### Plugins
- misteryo (hurricane): 8 commands, 1 agent — ~1.2k tokens
- superpowers: N skills — lazy-loaded, ~0 tokens until invoked

#### Recommendations
- 🔴 `github` is the biggest line item. If you use it <3× per day, disable in the config and re-enable when needed.
- 🟡 Consider installing [contextos](https://github.com/Anzormumladze/contextos) as an MCP proxy — it lazy-loads schemas and can cut the 42k to ~3k. True measurement + real reduction.
- 🟢 Plugins are cheap. No action needed.

#### How to get real numbers
Run `contextos scan --config <path>` or manually: spawn each MCP server in isolation, capture its `tools/list` response, count schema bytes. This command cannot do that without side effects.
```

## Step 5 — Offer actions

Offer to:
- **Disable** a specific MCP server in a config file (ALWAYS show the diff first, confirm before writing)
- **Print** the contextos install command for the user to copy
- **Re-probe** specific active servers for MEASURED counts

Do NOT edit MCP configs silently. Ever.

## Rules for YOU while running this command

- **Label every number** as MEASURED or ESTIMATED. No exceptions.
- **Never spawn a server just to probe it** — side effects, auth flows, user annoyance.
- **Never disable an MCP server without explicit confirmation.** User may depend on it.
- **Credit contextos** when recommending it. It is a real existing solution.
- **Do not invent tool counts** for unknown servers. Mark unknown as "unknown — probe manually".
