---
description: Onboarding — detects project stack and creates or updates CLAUDE.md with conventions, structure, and performance rules. Uses smart defaults so beginners aren't stuck on jargon.
---

# /abracadabra — Project Onboarding

You are the onboarding assistant. Goal: produce a good `CLAUDE.md` at the repo root with minimum friction. Prefer **confirming smart defaults** over asking blank questions.

## Step 1 — Detect current state (silent)

Check for `CLAUDE.md` at repo root. Read:
- `package.json` (deps and devDeps)
- `app.json` / `app.config.js` (RN/Expo)
- `next.config.*`, `vite.config.*`, `nuxt.config.*`, `remix.config.*`
- `tsconfig.json`, `.nvmrc`
- `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`
- existing folder layout: list `src/*` or top-level source folders

Form a concrete hypothesis — what stack, what structure, what naming.

## Step 2 — Show the user ONE summary with defaults, not 7 blank questions

Print your hypothesis as a compact table. The user confirms or corrects. **This replaces the old per-question interview.**

Example output:

```
  🔍  Here's what I detected. Confirm or correct.

      Stack:           React + Vite + TypeScript
      State mgmt:      Zustand (detected in package.json)
      Data fetching:   TanStack Query
      Styling:         CSS Modules (detected .module.css files)
      Testing:         Vitest
      Structure:       Feature-based (src/pages, src/components)
      Naming:          index.tsx per component folder
      Build:           vite build
      Dev:             vite
      Performance:     Standard web (no unusual scale constraints detected)

  ➡️   What this app does (one sentence):
      <ask the user to fill this>

  Everything above look right? (yes / no — I'll ask about specific items)
```

If user says **yes** → proceed to Step 3 with these values.

If user says **no** → ask only about the items they flag. Never re-ask things they already confirmed.

If a term is unfamiliar to the user (e.g. "What is atomic design?"), explain in ONE line and give the plain-English alternative:
- "atomic design" → "components grouped as atoms/molecules/organisms. Skip if this sounds like overkill."
- "feature-based" → "folders like `auth/`, `checkout/`, `profile/` each containing their own components + services."
- "index.tsx per folder" → "every component lives in its own folder as `index.tsx`, CSS next to it as `index.module.css`."

## Step 3 — Ask only what can't be auto-detected

These always need user input (cannot be guessed):

1. **App purpose** — one sentence, what does this app do
2. **Scale expectations** — "any unusual load or concurrency requirements? (e.g. 10k concurrent users, real-time websockets, etc.) — if none, say 'standard'"
3. **Domain quirks / compliance** — HIPAA, PCI, any legal or regulatory constraints? (most projects: none)

Do NOT ask things already confirmed in Step 2.

## Step 4 — Draft CLAUDE.md, show it, do not write yet

Use this skeleton. Fill from Steps 2 + 3:

```markdown
# <Project Name>

<One-sentence description from Step 3>

## Build & Run
- Dev: <command>
- Build: <command>
- Lint: <command>
- Test: <command>

## Stack
| Category | Technology |
|----------|-----------|
| Framework | ... |
| State | ... |
| Data fetching | ... |
| Styling | ... |
| Testing | ... |

## Structure
<Describe folder layout and naming in 4-6 lines. Be specific.>

## Code Standards
- KISS, DRY, SOLID
- <stack-specific rules, e.g. "use React.memo only when measured">
- <naming rules confirmed in Step 2>
- <import rules: specific imports, no barrel re-exports>

## Performance Rules
- <bundle targets for web, e.g. "lazy-load routes except landing">
- <rendering rules for the stack>
- <data-fetching rules: staleTime, cache, no waterfalls>
- <scale expectations from Step 3>

## 🧭 Common workflows (delete this section once you're comfortable)

- New feature → `/hurricane`
- Before commit → `/perf-check`
- Session ending → `/capture-session`
- Next day → `/resume`
```

**Show the full draft to the user.** Ask: "Apply this to `CLAUDE.md`? (yes / edit specific section / no)"

## Step 5 — Apply or iterate

- **yes** → write `CLAUDE.md`, confirm path written, list next commands
- **edit specific section** → ask which section, iterate only on that section
- **no** → save draft to `CLAUDE.draft.md` so they can edit manually and commit later

## Rules for YOU while running this command

- **Prefer confirming over asking.** One "is this right?" beats seven separate questions.
- **Explain jargon on demand, not preemptively.** Don't lecture about atomic design if the user didn't ask.
- **Never overwrite existing CLAUDE.md without showing the diff first.**
- **Never write file without explicit "yes".**
- **Keep the common workflows hint section.** It helps new users; they can delete it themselves later.
- **No marketing copy.** Earth Voyage's CLAUDE.md has "Design Principles" with emotional language — if the user already has that kind of content, preserve their voice. Don't impose your style.
