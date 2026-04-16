---
description: Validates current changes against structural rules defined in CLAUDE.md — folder layout, naming conventions, imports, and component organization.
---

# /structure-check — Structure Validator

Validate that recent changes (or the whole repo, if no diff) follow the structural rules defined in `CLAUDE.md`.

## Step 1 — Load the rules

Read `CLAUDE.md` at the repo root. Extract:
- Folder structure rules (where components, services, stores, etc. go)
- Naming conventions (file names, component names, CSS module names)
- Import rules (barrel files? specific imports? no wildcard?)
- Component organization (one folder per component? `index.tsx` pattern?)

If `CLAUDE.md` does not exist or has no structure section, tell the user to run `/abracadabra` and stop.

## Step 2 — Determine scope

Default scope: uncommitted changes + changes vs `main` branch.

Run:
```bash
git diff --name-only HEAD
git diff --name-only main...HEAD
```

If the user passed a path argument, scope to that path instead.

## Step 3 — Audit each changed file

For every changed file, check:

1. **Location** — is it in the right folder per the rules?
2. **Filename** — matches the naming convention?
3. **Component folder pattern** — if the rule says "one folder per component with `index.tsx`", does the file follow it?
4. **Imports** — no barrel imports if disallowed; specific imports; no cross-layer violations (e.g. `pages/` importing from `pages/`)?
5. **Colocation** — CSS modules next to components, tests next to source, etc.?

## Step 4 — Report

Output a categorized report:

```
### Structure Issues Found

#### 🔴 Violations (must fix)
- `src/foo/bar.tsx` — should be `src/components/bar/index.tsx` per convention
- `src/components/Card.tsx` — missing folder; convention is one folder per component

#### 🟡 Warnings (consider fixing)
- `src/utils/helpers.ts` — imports from `src/components/*` (wrong direction)

#### ✅ Clean
- 12 files pass all checks
```

## Step 5 — Offer to fix

For each 🔴 violation, offer to fix it. Do NOT auto-apply fixes — always confirm with the user per-file (or ask "fix all?" once).

## Rules for YOU while running this command

- Base every finding on an explicit rule from `CLAUDE.md`. If a "violation" has no rule behind it, don't flag it.
- Do NOT flag stylistic preferences that aren't in `CLAUDE.md` — structure-check is about structure, not taste.
- Be concise. List violations, don't lecture.
