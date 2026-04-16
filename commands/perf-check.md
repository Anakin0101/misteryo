---
description: Performance audit — bundle size, rendering, network, and 10k-user scale scenarios. Scans code and reports actionable issues.
---

# /perf-check — Performance Audit

Audit the project (or a scoped path) for performance issues across four dimensions: **bundle**, **rendering**, **network**, and **scale**.

## Step 1 — Load performance rules

Read `CLAUDE.md` at the repo root. Extract the "Performance Rules" section. If absent, use sensible defaults for the detected stack.

## Step 2 — Determine scope

Default: changed files vs `main` branch. If the user passed a path, scope to that path.

## Step 3 — Audit in parallel

Spawn four independent `Explore` subagents (parallel — in one message) to cover each dimension. Each returns a short findings report.

### Bundle audit
Look for:
- Heavy libraries imported statically that should be `React.lazy()` / dynamic `import()` (e.g. map libs, PDF libs, carousels, charting libs)
- Barrel file imports pulling in more than needed
- Duplicate dependencies
- Large static assets (>100KB images) not in WebP/AVIF
- Missing route-level code splitting

### Rendering audit (React/RN projects)
Look for:
- Inline objects/arrays/functions in JSX props (`style={{}}`, `onClick={() => {}}`) on hot paths
- `.filter` / `.map` / `.reduce` / `.sort` chains in render without `useMemo`
- Array index used as `key` in `.map()`
- Missing `React.memo` on pure children of frequently re-rendering parents
- Context providers whose value is a new object every render
- Zustand stores subscribed as a whole instead of selectively

### Network audit
Look for:
- TanStack Query / RTK Query hooks without `staleTime` (transient defaults = refetch storms)
- Request waterfalls (chained `enabled` flags that serialize independent fetches)
- Mutations that don't invalidate related queries
- `fetch` calls inside components without abort on unmount
- N+1 patterns (loop calling an API per item)
- Missing pagination on lists that can grow unbounded

### Scale audit (what breaks at 10k concurrent users?)
Reason about:
- Client-side work that should be server-side (filtering huge lists, heavy computation)
- Unbounded localStorage / sessionStorage writes
- WebSocket connections without reconnect/backoff
- Global intervals/timers that never clean up
- Analytics events fired in tight loops
- Images loaded without CDN / cache headers consideration

## Step 4 — Report

Consolidate findings into one report with severity and fix effort:

```
### Performance Audit

#### 🔴 Critical (ship-blockers)
- `src/pages/home/index.tsx:42` — static import of `maplibre-gl` (~200KB) on landing route; move to `React.lazy()`
  **Fix effort:** 15 min

#### 🟠 High
- `src/components/feed/index.tsx:88` — inline callback in `.map()` causing child re-renders; extract to `useCallback`

#### 🟡 Medium
- `src/service/posts.ts:12` — `useQuery` without `staleTime`; recommend 5 min for user data

#### ✅ Clean
- No rendering issues in `src/components/card/`
- No scale red flags in current changes
```

## Step 5 — Offer fixes

For 🔴 and 🟠 issues, offer to apply fixes with the `perf-lead` agent if it's available in this project, or directly. Confirm per-issue or ask "fix all critical?" once.

## Rules for YOU while running this command

- **Evidence-based.** Every finding must cite file + line.
- **No speculation.** If you're not sure an issue exists, say "possible issue, needs verification" — don't report as fact.
- **Respect CLAUDE.md.** If the user's rules say "no React.memo unless measured", don't flag missing memo.
- **Be brief.** Each finding = one line describing the issue + one line with the fix.
