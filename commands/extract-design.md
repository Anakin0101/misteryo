---
description: Extract a design system (colors, fonts, spacing, tokens) from any live website using designlang. Wraps the designlang CLI.
---

# /extract-design — Pull design tokens from any URL

Wraps [designlang](https://github.com/Manavarya09/design-extract) by Manav Arya — a CLI tool that reverse-engineers design systems from live websites using Playwright.

Use this when you want to:
- Capture tokens from a reference/inspiration site
- Compare your site's design to a competitor
- Score your own site's design quality (A–F)
- Generate a starter config (Tailwind, shadcn theme, W3C tokens) from a URL

## Inputs

The user provides a URL (or multiple for comparison). Optional flags described below.

## Step 1 — Confirm intent

Ask the user which mode they want, unless already obvious:

1. **Extract** — full token dump (`npx designlang <url>`)
2. **Apply** — auto-detect framework in current project and write tokens (`npx designlang apply <url> --dir .`)
3. **Clone** — generate a Next.js starter app mirroring the site (`npx designlang clone <url>`)
4. **Score** — rate the design A–F (`npx designlang score <url>`)
5. **Compare brands** — diff multiple sites (`npx designlang brands <url1> <url2> ...`)

## Step 2 — Warn about cost

Before running, tell the user:
- First run downloads Playwright (~200MB, one-time)
- Each extraction takes 30s–2min depending on site size
- Output is written to the current directory unless `--dir` specified

Confirm they want to proceed.

## Step 3 — Run the command

Execute the chosen `npx designlang ...` invocation via Bash. Stream output.

### Error paths — diagnose, don't retry blindly

| Symptom | Likely cause | What to tell the user |
|---------|-------------|----------------------|
| `node: command not found` or `EBADENGINE` | Node < 18 | "designlang requires Node 18+. Check `node --version`. Upgrade via nvm: `nvm install 20`." |
| `ENOTFOUND` / network timeout | Offline or DNS issue | "Network error reaching the URL or npm. Check connectivity, try again." |
| `Protocol error` / Playwright crash on launch | Chromium download failed or blocked | "Playwright install may have been interrupted. Try: `npx playwright install chromium`." |
| `ERR_SSL_PROTOCOL_ERROR` / cert issues | Corporate proxy or self-signed cert | "SSL error hitting the target URL. Is this behind a VPN or proxy? Try a public URL first." |
| `Page.goto: Timeout` | Target site very slow or blocks headless browsers | "Site didn't load in time. Some sites (Cloudflare-protected) block headless browsers. Try a different URL." |
| `Permission denied` writing output | Working directory not writable | "Output directory not writable. Change to a writable directory and retry." |

**Do not retry the same command after a hard failure.** Report the diagnosis and wait for user input.

## Step 4 — Summarize output

designlang writes up to 8 files (markdown, HTML preview, W3C tokens JSON, Tailwind config, React theme, Figma Variables, CSS vars, shadcn theme). List what was created and where.

If mode was `score`, show the A–F grade and the breakdown.

If mode was `apply`, show the diff of token files that changed in the current project. Do NOT commit.

## Step 5 — Offer next steps

Based on mode:
- **Extract** → "Want me to review the tokens against your current CLAUDE.md design rules?"
- **Apply** → "Run `/structure-check` to validate the new tokens fit your conventions?"
- **Score** → "Want me to highlight the weakest dimensions and suggest fixes?"
- **Compare** → "Summarize the biggest differences between the sites?"

## Rules for YOU while running this command

- **Don't invent URLs.** Only run on URLs the user explicitly provided.
- **Don't auto-commit.** designlang writes files; leave git decisions to the user.
- **If Playwright download fails**, say so and link https://playwright.dev/docs/intro — do not try workarounds.
- **Respect the existing project.** In `apply` mode, show diffs before overwriting any existing token file.
- **Credit designlang** when showing output — it's Manav Arya's tool, hurricane just wraps it.
