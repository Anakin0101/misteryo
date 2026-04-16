---
description: Optimize CLAUDE.md to reduce token overhead — removes duplication, verbosity, and low-value sections while preserving rules and structure.
---

# /compress-context — CLAUDE.md Optimizer

Reduce the token cost of `CLAUDE.md` without losing its signal.

Inspired by [contextos](https://github.com/Anzormumladze/contextos) by Anzor Mumladze — a standalone MCP proxy for broader token reduction. This command focuses narrowly on `CLAUDE.md`.

## Why this matters

`CLAUDE.md` is loaded into context on every turn. A 10KB file costs ~2500 tokens per turn. Over a long session with many tool calls, that compounds fast. Trimming fluff is a free speedup.

## Step 1 — Read and measure

Read `CLAUDE.md` at the repo root. Report current size in lines and approximate tokens (rule of thumb: 1 token ≈ 4 chars of English, ≈ 3 chars for code).

### Error paths
- **No `CLAUDE.md`** → "No CLAUDE.md found. Run `/abracadabra` first to create one." Stop.
- **CLAUDE.md is empty (0 bytes)** → "CLAUDE.md exists but is empty. Run `/abracadabra` to rebuild it." Stop.
- **CLAUDE.md < 500 bytes** → "CLAUDE.md is very small (~<N> tokens). Compression won't help — it's already minimal. Skipping." Stop without writing anything.
- **CLAUDE.md not valid UTF-8 / unreadable** → report the specific error, suggest checking file encoding. Stop.

## Step 2 — Identify compression targets

Scan for:

1. **Duplication** — the same rule stated twice in different sections
2. **Verbose prose** — "It is important to note that when you..." → "When..."
3. **Examples that don't add information** — example code that just restates the rule
4. **Low-signal sections** — exhaustive dependency lists that can be derived from `package.json`
5. **Historical notes** — "we used to do X but now..." (belongs in git history, not CLAUDE.md)
6. **Markdown bloat** — nested bullet lists 4 levels deep, redundant headers
7. **Instructions for humans** — onboarding text meant for developers, not Claude

## Step 3 — Produce a compressed version

Write the optimized `CLAUDE.md` to a temp location (or show as diff). Follow these principles:

- **Tables beat prose** for structured info (stack, conventions)
- **One line per rule** when possible
- **Keep the "why"** only when it's non-obvious — otherwise just state the rule
- **Consolidate** scattered rules into single sections
- **Preserve every actual rule** — compression ≠ deletion. If a rule existed, it must still exist.

## Step 4 — Show diff and stats

Present:
```
Before: 342 lines, ~2800 tokens
After:  198 lines, ~1450 tokens
Saved:  ~1350 tokens per turn (~48%)
```

Show a unified diff. Highlight anything that looks like content loss (not just formatting).

## Step 5 — Confirm and apply

Ask the user: "Apply this compressed version?" — and only overwrite on explicit yes.

## Rules for YOU while running this command

- **Never silently drop rules.** If you remove something, call it out explicitly: "Removed X because Y — confirm?"
- **Don't change meaning.** Rewording must preserve intent exactly.
- **Don't compress past the point of clarity.** A 50% smaller file that Claude misreads is worse than the original.
- **Leave the user a rollback.** Keep the original as `CLAUDE.md.bak` on apply.
