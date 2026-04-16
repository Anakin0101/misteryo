---
description: Save a structured summary of the current session to memory/sessions/ so /resume can continue the work later. Run at the end of a working session.
---

# /capture-session — Save Session Summary

Write a structured summary of **this conversation** to the Claude Code memory folder so `/resume` can pick up from here in a future session.

## What this does (honest scope)

This does NOT save the full conversation — that would be huge and low-signal. It extracts the parts that actually matter for continuation:
- What task was being worked on
- Decisions made (and why)
- Files touched
- What's done vs what's still open
- Blockers or open questions

## Step 1 — Locate memory folder

Find the Claude Code memory folder for this project:
- `~/.claude/projects/<project-path-slug>/memory/`

Create `memory/sessions/` if it does not exist. Do NOT write to `MEMORY.md` or other existing memory files — those are for persistent cross-session memory, different purpose.

If the memory folder itself doesn't exist, this is likely a very new Claude Code session. Say so and stop — no session summary to capture yet.

## Step 2 — Decide if there's anything worth capturing

Look at the conversation. If the session contained only:
- Casual chit-chat with no real decisions or code changes
- A single read-only question answered
- Nothing that would help a future session

...then say so honestly: "This session doesn't have enough substance to capture. Skipping." and stop.

Only capture when there's real signal — decisions, code changes, blockers, or non-trivial discussion.

## Step 3 — Ask the user for a title (quick)

One question: "What's the best title for this session? (3-8 words, something you'll recognize later)"

Skip this only if the title is obvious from the conversation and the user hasn't pushed back on your inference.

## Step 4 — Extract the summary from conversation

Look back through the conversation. Build the summary using ONLY facts from what actually happened. Do not invent or infer beyond what was discussed.

Fill this template:

```markdown
---
title: <user-provided or inferred title>
date: <YYYY-MM-DD HH:MM>
project: <repo name>
branch: <current git branch>
---

# Session: <title>

## Task
<1-2 sentences — what the user was trying to do>

## Decisions made
- <decision 1 — with the *why* if discussed>
- <decision 2>
- ...

## Files touched
- `path/to/file1.ext` — <one-line what changed>
- `path/to/file2.ext` — <one-line>

## Status
- ✅ Done: <what's finished>
- 🚧 In progress: <what's partially done, with state>
- ❌ Blocked: <what stopped us, and why>

## Open questions
- <question 1>
- <question 2>

## Next step
<one sentence — what the user or next session should do first>
```

## Step 5 — Write the file

Filename: `session-YYYY-MM-DD-HHMM.md` (use current date/time).

Write to `<memory-folder>/sessions/session-YYYY-MM-DD-HHMM.md`.

## Step 6 — Confirm

Show the user:
- The full path where it was written
- A 3-line preview (title + task + next step)
- Reminder: "Run `/resume` in a new session to continue from here."

## Automation tip (optional, for the user)

Mention once (only on the user's first capture): Claude Code supports SessionEnd hooks in `~/.claude/settings.json`. A hook command like `claude run /capture-session` on session end would automate this — but it requires manual settings.json config. Skip if the user isn't interested.

## Rules for YOU while running this command

- **Only summarize what actually happened in this conversation.** Do not pad with assumptions or generic advice.
- **Be honest about unfinished state.** If a task is half-done, say exactly which half.
- **Keep it short.** A good session summary is ~30 lines. If yours is 100 lines, you're including too much.
- **Don't write code suggestions into the summary.** The summary is a log, not a plan. Next steps = 1 sentence max.
- **Never overwrite an existing file silently.** If the filename collides (unlikely), append `-2`, `-3`, etc.
- **Respect privacy.** Don't dump raw API keys, tokens, or secrets the user pasted — redact them as `<redacted>`.
- **Don't force a capture.** If the session has no real content (Step 2), decline to capture.
