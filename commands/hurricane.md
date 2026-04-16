---
description: Adaptive dev pipeline — classifies task size and runs minimal / standard / full flow. Small fix = direct edit. Feature = review-gated flow. Refactor = full 6-stage pipeline.
---

# /hurricane — Adaptive Dev Pipeline

You are running the Hurricane pipeline. **First classify the task size, then match the depth to it.** Do not force a 6-stage flow on a typo fix.

## Preflight checks (run these first, fail loudly if broken)

1. **CLAUDE.md exists?** If missing → stop, tell user to run `/start` (or `/abracadabra`).
2. **CLAUDE.md non-empty and readable?** If empty or unparseable (e.g. 0 bytes, or not valid markdown) → stop, tell user to run `/abracadabra` to rebuild it.
3. **Required superpowers skills available?** This command uses `superpowers:brainstorming`, `superpowers:writing-plans`, and `superpowers:requesting-code-review`. If any is missing:
   - **brainstorming / writing-plans missing** → fall back gracefully: do the stage inline (ask the user directly, write the plan yourself) and note "superpowers plugin not installed — running in basic mode".
   - **requesting-code-review missing** → do an inline review yourself instead of spawning the skill.
   - Do NOT crash the pipeline because a skill is missing. Degrade, don't fail.
4. **performance-reviewer agent available?** If this plugin is installed, yes. If not (somehow missing), do the perf review inline using the rules from the agent's prompt.

Announce degradations clearly: "Note: `superpowers` skills not installed. Running brainstorm/plan inline. Install `superpowers` for the full experience."

## Step 1 — Classify the task

Ask the user (or infer from their initial message):

```
  📋  Before I start — what size is this task?

      1. 🐣  Tiny      Typo, single-line fix, rename, obvious bug
                       → I'll just do it. No pipeline.

      2. 🛠️   Small     Add a component, new API endpoint, small refactor
                       → Minimal flow: plan → implement → review

      3. 🌀  Medium    New feature, multi-file change, UI flow
                       → Standard flow: brainstorm → plan → implement → review → perf

      4. 🏗️   Large     Refactor, migration, architecture change
                       → Full flow: all 6 stages with checkpoints

      Which? (1 / 2 / 3 / 4)  or describe the task and I'll pick.
```

If the user describes the task rather than picking a number, classify silently using these rules:
- **Tiny** — one-line change, no logic shift, no tests needed
- **Small** — 1-3 files, isolated scope, no architectural impact
- **Medium** — multiple files, crosses layers, new user-facing behavior
- **Large** — touches core architecture, requires migration, or rewrites existing systems

State your classification: "Classifying as **Medium** because touches 4 files and introduces new user flow. Correct?" — let the user override.

## Step 2 — Route to the matching flow

### 🐣 Tiny flow (no pipeline)

Just do the task. No brainstorm, no plan doc, no formal review. After the edit:
- Show the diff
- Ask: "Looks good? Want me to run `/perf-check` on the changed file before commit?"

**Skip to end.** Do not force review on a typo.

### 🛠️ Small flow (3 stages)

1. **Plan** — one paragraph, what files, what approach. Confirm with user (fast).
2. **Implement** — execute.
3. **Review** — invoke `superpowers:requesting-code-review` on the diff.

No brainstorm, no full perf-review (unless review flags perf concerns). User confirmation once at end of plan, once at end of review.

### 🌀 Medium flow (5 stages)

1. **Brainstorm** — `superpowers:brainstorming`. Short — 2-3 approaches, tradeoffs.
2. **Plan** — `superpowers:writing-plans`.
3. **Implement** — follow CLAUDE.md rules strictly.
4. **Code review** — `superpowers:requesting-code-review`.
5. **Performance review** — spawn `performance-reviewer` agent.

Checkpoints after each stage.

### 🏗️ Large flow (full 6 stages)

All 5 from Medium, plus:

6. **Post-review summary** — what changed, why, risks, follow-ups the user should track separately.

Checkpoints mandatory. Slow is fine — large changes earn the ceremony.

## Step 3 — Implement-stage rules (apply to Small / Medium / Large)

Read `CLAUDE.md`. Follow its standards:
- Reuse existing components/utilities before creating new ones
- Match naming, folder, and import conventions
- KISS, DRY, SOLID — but no over-engineering
- No inline `style={{}}` / `onClick={() => {}}` in JSX on hot paths
- Stable keys in `.map()`
- Memoize expensive computations

Use `TaskCreate` to track progress for Medium and Large. Skip task tracking for Tiny and Small — overhead.

## Step 4 — End-of-flow summary

All flows end with:

```
  ✅  Done

  What changed:
    - <file1>: <1-line description>
    - <file2>: ...

  Stage summary:
    Brainstorm  | <brief or "skipped (tiny/small)">
    Plan        | <brief or "skipped">
    Implement   | done
    Review      | clean / fixed N issues
    Perf        | <brief or "skipped">

  Ready to commit? (I will not commit without your explicit "yes")

  💡  Before you close Claude Code, run /capture-session
      so tomorrow's /resume knows where you left off.
```

## Rules for YOU

- **Match depth to task.** Do not run 6 stages on a typo. Do not skip brainstorm on a refactor.
- **State your classification openly.** "This looks Small because X." User can override.
- **Skip tasks intelligently.** For Tiny, no `TaskCreate`, no plan doc. For Large, every stage tracked.
- **Checkpoints are mandatory for Medium/Large.** Do not run stage N+1 until user confirms stage N.
- **If CLAUDE.md is missing**, stop. Redirect to `/start`.
- **Never `git commit`** unless the user explicitly says so.
- **Never escalate size silently.** If a Small task is growing into Medium mid-flow, stop and reclassify with the user.
