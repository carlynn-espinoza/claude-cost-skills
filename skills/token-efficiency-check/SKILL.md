---
name: token-efficiency-check
description: Final efficiency review before building any agent, scheduled task, cron, or automation loop. Trigger right before creating, committing, or scheduling a new agent/task/cron/loop, or when materially changing an existing one. Catches redundant runs, oversized models, context/scope bloat, and bad cadence. Run as the last gate before Claude takes build actions. Do NOT use to review ordinary back-and-forth chat.
---

# Token Efficiency Check

A pre-build gate. Run this **once, right before** building, committing, or scheduling an agent, scheduled task, cron, or automation loop, not during the planning conversation. The goal is keeping cost as close to $0 as possible without losing the result. Output is value, not runtime or tokens spent.

## When to run

Run this review when about to:
- create or schedule a new agent, scheduled task, cron, or automation loop
- materially change what an existing one does, how often it runs, or what it reads/writes

Do **not** run it on normal chat, planning, or read-only questions.

## How to run

1. Read the final spec of the thing being built: trigger/cadence, which model it calls, what data/files/context it pulls, what it writes, and how it exits on no-op.
2. Walk the four checks below. For each, mark PASS or FLAG with a one-line reason.
3. If everything passes, say so in one line and proceed to build.
4. If anything flags, stop and surface the flags as a short list with a concrete fix for each, then ask before building. Do not silently "fix and proceed."

Keep the report short. No essays.

## The four checks

### 1. Redundant / duplicate runs
- Does an existing agent or scheduled task already do this or overlap with it? List the overlap.
- If two paths can produce the same artifact, is there a dedup guard so it can't double-write (the blog double-post lesson)?
- Could this be folded into an existing task instead of a new one?

### 2. Model right-sizing
- Does this step need an LLM at all, or can plain code / an API / a template do it deterministically and free?
- If it needs an LLM, is it on the cheapest model that still produces the result? Default to Haiku and only step up with a stated reason.
- Are expensive calls batched rather than looped one-item-at-a-time?

### 3. Scope / context bloat
- Does it pull only the data, files, and fields it actually needs, or grab whole datasets/folders "just in case"?
- Is reusable input cached or read once, not re-fetched every iteration?
- Are output length and runtime capped so a runaway can't burn tokens? No unbounded loops.
- Does it early-exit cleanly when there's nothing to do (no-op should be cheap)?

### 4. Cadence / frequency
- Does run frequency match how often the underlying data actually changes? Daily when weekly would do is waste.
- Is it idempotent and safe to re-run, with sane failure handling so retries don't pile up cost?

## Output format

```
Token efficiency check — <thing being built>
1. Redundant runs:   PASS / FLAG — reason
2. Model sizing:     PASS / FLAG — reason
3. Scope/context:    PASS / FLAG — reason
4. Cadence:          PASS / FLAG — reason

Verdict: clear to build  |  fixes needed (see below)
```

If fixes are needed, list each flag with one concrete fix and wait for a go-ahead.
