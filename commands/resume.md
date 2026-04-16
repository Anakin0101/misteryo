---
description: Resume the last session — loads the most recent session summary from the memory folder and picks up where you left off.
---

# /resume — Session Continuation

Load the most recent session summary from Claude Code's memory folder and continue work.

## Honest scope

This does **not** restore the full previous conversation — that's technically impossible and would blow the context window anyway. What it does:

1. Loads the most recent session summary from `memory/sessions/`
2. Replays the key decisions, current task state, and open questions into your context
3. Lets you continue with the essential facts, not the full transcript

For most real work (decisions, current task, blockers), this is what you actually need.

## Step 1 — Find memory folder

Locate the Claude Code memory folder for this project. Typically:
- `~/.claude/projects/<project-path-slug>/memory/`

If the folder doesn't exist, tell the user: "No prior session summaries found. Session summaries are written when you end a conversation — nothing to resume yet."

## Step 2 — Find the most recent session summary

Look in `memory/sessions/` for files matching `session-YYYY-MM-DD-HHMM.md`. Pick the most recent.

If none exist (only the standard `MEMORY.md` and persistent memories are there), tell the user that sessions haven't been captured yet and suggest they run `/capture-session` before ending their next conversation. (Note: `/capture-session` is a future command — for now, the user can manually save a summary.)

## Step 3 — Load and summarize

Read the session summary file. Extract:
- What task was in progress
- What decisions were made
- What files were touched
- What's still open / blocked

Present this back to the user in ~10 lines max:

```
### Resuming from session 2026-04-16 18:32

**Task in progress:** Fixing keyboard suppression in WriteMessageModal (PD-717)
**Last decisions:**
- Scope narrowed to iOS Safari only
- Landed on `inputmode="none"` + readOnly fallback

**Files touched:**
- src/components/write-message-modal/index.tsx
- src/components/write-message-modal/index.module.css

**Open questions:**
- Does fix behave correctly with VoiceOver? (untested)

**Next step per last session:** Run manual test on iOS 17 Safari.
```

## Step 4 — Confirm and hand back

Ask: "Continue this task, or start something new?"

## Rules for YOU while running this command

- **Do not pretend to remember the full conversation.** You don't. Work from the summary only.
- **Verify before acting.** If the summary says "file X was renamed to Y", check that Y actually exists before assuming it.
- **If the summary is stale** (older than a week, or the code has clearly moved on), say so and ask the user to confirm what's still relevant.
