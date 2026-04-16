# 🌀 Misteryo — Hurricane Plugin for Claude Code

An opinionated workflow plugin for [Claude Code](https://claude.com/claude-code). Sets up project conventions, runs full dev pipelines, and enforces structure and performance standards defined in your `CLAUDE.md`.

Works across any stack — React, React Native, Next.js, Vue, Go, Python, Rust. The plugin adapts by reading your project's `CLAUDE.md`.

---

## ⚡ Quick start (2 minutes)

```
1. Install:            /plugin install AkakiNeparidze/misteryo
2. First thing:        /start
3. Your first task:    /hurricane
4. Before commit:      /perf-check
5. Session ending:     /capture-session
```

**Everything else is optional.**

`/start` is the entry point — it shows a welcome banner, detects your project, and launches `/abracadabra` (a 3-minute interview that writes a tailored `CLAUDE.md`) if needed.

---

## Commands

### 🎯 Core — memorize these

| Command | When to use |
|---------|-------------|
| `/start` | **Run first** after install. Welcome banner, project detection, guides you into setup. |
| `/hurricane` | Any feature or bugfix. Runs brainstorm → plan → code → review → perf, with checkpoints. |
| `/perf-check` | Before every commit. Audits bundle, rendering, network, and scale. |
| `/capture-session` | Before closing Claude Code. Saves a structured summary to memory. |
| `/resume` | When reopening Claude Code. Loads the last session summary. |

### 🧰 Extra — use when needed

| Command | What it does |
|---------|-------------|
| `/abracadabra` | Onboarding interview. Run after stack changes or to re-setup `CLAUDE.md`. |
| `/structure-check` | Validates changed files against structure rules in `CLAUDE.md`. |
| `/compress-context` | Slims `CLAUDE.md` to reduce per-turn token cost. |
| `/scan-tokens` | Audits MCP servers and plugins for session-start token overhead. |
| `/extract-design` | Pulls design tokens from any live website (wraps [designlang](https://github.com/Manavarya09/design-extract)). |

### 🤖 Agent

- `performance-reviewer` — stack-aware perf reviewer. Detects your stack (React, Go, Python, etc.) and applies only relevant checks. Spawned by `/hurricane` and `/perf-check`.

---

## Design principles

- **One plugin, many stacks.** The plugin reads `CLAUDE.md` and adapts. You don't fork per-project.
- **Checkpoints over automation.** `/hurricane` pauses at each stage for user approval. No runaway agents.
- **Evidence over speculation.** Every finding cites `file:line`. No vague warnings.
- **Honesty over numbers.** `/scan-tokens` labels every token count MEASURED or ESTIMATED. No fake precision.
- **Credit where due.** `/compress-context`, `/scan-tokens` inspired by [contextos](https://github.com/Anzormumladze/contextos) by Anzor Mumladze. `/extract-design` wraps [designlang](https://github.com/Manavarya09/design-extract) by Manav Arya.

---

## Typical workflows

**New project**
```
/start → (auto-launches) /abracadabra → /hurricane
```

**Existing project, new feature**
```
/hurricane → /perf-check → commit → /capture-session
```

**Next day**
```
/resume → /hurricane → /perf-check → /capture-session
```

**Performance concerns**
```
/scan-tokens → /compress-context → /perf-check
```

**Design reference**
```
/extract-design score <your-url>       (where are you)
/extract-design extract <reference-url> (what's good out there)
```

---

## Adding your own commands

Drop a new `.md` file into `commands/` with this frontmatter:

```markdown
---
description: One-line description shown in the slash-command menu.
---

# /your-command

Instructions, step by step.
```

Push to GitHub. Users run `/plugin update misteryo` and get the new command.

---

## License

MIT
