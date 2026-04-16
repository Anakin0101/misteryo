---
description: Welcome + onboarding. Run this first after installing the plugin. Shows a banner, detects project state, and guides the user to /abracadabra or the core commands.
---

# /start — Welcome & Onboarding

First command a user should run after installing Misteryo. Shows a welcome banner and guides them into the correct next step.

## Step 1 — Print the welcome banner (verbatim)

Output this exact banner as your first message. Keep the ASCII art intact, do not substitute emojis with text or vice versa:

```

  ╔═══════════════════════════════════════════════════════════════════════════╗
  ║                                                                           ║
  ║   ███╗   ███╗██╗███████╗████████╗███████╗██████╗ ██╗   ██╗ ██████╗        ║
  ║   ████╗ ████║██║██╔════╝╚══██╔══╝██╔════╝██╔══██╗╚██╗ ██╔╝██╔═══██╗       ║
  ║   ██╔████╔██║██║███████╗   ██║   █████╗  ██████╔╝ ╚████╔╝ ██║   ██║       ║
  ║   ██║╚██╔╝██║██║╚════██║   ██║   ██╔══╝  ██╔══██╗  ╚██╔╝  ██║   ██║       ║
  ║   ██║ ╚═╝ ██║██║███████║   ██║   ███████╗██║  ██║   ██║   ╚██████╔╝       ║
  ║   ╚═╝     ╚═╝╚═╝╚══════╝   ╚═╝   ╚══════╝╚═╝  ╚═╝   ╚═╝    ╚═════╝        ║
  ║                                                                           ║
  ║          🌀  Hurricane workflow plugin for Claude Code                    ║
  ║                                                                           ║
  ╚═══════════════════════════════════════════════════════════════════════════╝

  👋  Welcome! Let me set you up.

```

## Step 2 — Detect project state

Silently (do not narrate every step) check:

1. Is there a `CLAUDE.md` at the repo root?
2. Peek at `package.json`, `go.mod`, `pyproject.toml`, `Cargo.toml`, `Gemfile`, `app.json` to guess the stack
3. Is `git` initialized? (optional — only matters if you plan to mention branching)

## Step 3 — Branch based on state

### Branch A — No CLAUDE.md exists

Print:

```
  📋  Project state
      • Stack detected: <your best guess, e.g. "React + Vite + TypeScript">
      • CLAUDE.md:      ❌  not found

  ⚡  Next step
      CLAUDE.md is how this plugin learns about your project's rules.
      Without it, other commands don't know how to help you.

      I'll launch /abracadabra now — a short interview that creates
      CLAUDE.md tailored to your stack. It takes ~3 minutes.

  Ready to start the interview? (yes / not now)
```

If the user says **yes**: invoke `/abracadabra`. Do not re-ask questions `/abracadabra` will ask.

If the user says **not now**: print:
```
  👍  No problem. Run /abracadabra whenever you're ready.
      Most commands need CLAUDE.md to work properly.
```
Then stop.

### Branch B — CLAUDE.md exists

Print:

```
  📋  Project state
      • Stack detected: <best guess>
      • CLAUDE.md:      ✅  found

  🎯  Your 4 core commands

      🌀  /hurricane         For every feature or bugfix (full pipeline)
      ⚡  /perf-check        Performance audit before commit
      💾  /capture-session   Save session summary before closing
      🔄  /resume            Load the last session summary

  🧰  Extra commands (use when needed)

      🏗️   /structure-check   Validate structure vs CLAUDE.md rules
      📉  /compress-context   Slim CLAUDE.md to reduce token cost
      📊  /scan-tokens        Audit MCP servers' token overhead
      🎨  /extract-design     Pull design tokens from any website
      ✨  /abracadabra        Re-run onboarding (e.g. after stack change)

  💡  Tip
      Most sessions go: /resume → /hurricane → /perf-check → /capture-session

  What would you like to do first?
```

Then wait. Do not auto-invoke anything.

## Step 4 — Stack warnings (only if relevant)

If the detected stack is something Misteryo hasn't been tested on (e.g. Elixir, Haskell, embedded C), add a gentle note:

```
  ⚠️   Heads up: your stack (<name>) is less common for this plugin.
      The performance-reviewer agent may apply fewer stack-specific
      checks. Core commands (hurricane, structure-check) work everywhere.
```

## Rules for YOU while running this command

- **Print the banner exactly once**, as the first thing the user sees. Do not print it after `/abracadabra` returns.
- **Do not repeat information.** If CLAUDE.md exists, do not explain what `/abracadabra` does unless the user asks.
- **Do not nag.** If the user says "not now", accept it. Do not re-prompt.
- **Do not run any other command automatically** beyond `/abracadabra` in Branch A (and only with user consent).
- **Keep it visually clean.** Preserve spacing, alignment, box-drawing characters. Terminals that hide emojis will still show the ASCII structure.
- **Do not guess the stack aggressively.** If uncertain, say "Stack: likely <X>, not confirmed" — don't commit to a wrong guess.
