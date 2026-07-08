---
name: check-cost
description: Cost-first working check. Trigger BEFORE any expensive step — generating full content (blogs, long copy, summaries), image generation, looping an LLM over many items, bulk writes to Sanity/Notion, or large fetches/builds. Enforces cheapest-path-first — run the cheap check that could make the expensive work unnecessary or smaller (dedup / overlap / existence / feasibility) BEFORE paying for it, and prefer free deterministic methods (a query, plain code, a template, a cache) over a model call. Do NOT use for the design-time review of a new agent/cron/loop — that is token-efficiency-check.
---

# Check Cost (Cheapest Path First)

The goal is always to reach the same result for as close to $0 as possible. The way you do that is **ordering**: do the cheap thing that could kill or shrink the expensive thing *before* you do the expensive thing. Never produce an expensive artifact and then throw it away.

## When to run

Right before you're about to spend real cost:
- generate a full artifact — a blog post, long-form copy, an AI summary, an OG image
- call an LLM in a loop over many items
- bulk-write to Sanity or Notion
- fetch or process a large dataset, or run a heavy build

Skip it for cheap read-only questions and normal chat.

## The move

Before the expensive step, ask: **"What cheap check could make this unnecessary, smaller, or better-targeted?"** Run that check first. Only pay for the expensive step on what survives.

The canonical lesson — blog generation:
- ❌ Write three full articles, then discard the ones that duplicate existing posts. You paid to generate throwaway work.
- ✅ Compare the *proposed topics* (cheap: a list + a GROQ/Notion query against existing titles/clusters) first. Drop or replace the overlapping topics. Then write full articles only for the ones that survived.

Same result, a fraction of the cost, because the cheap signal (topic) gated the expensive work (the article).

## Decision steps

1. **Name the expensive step.** What here actually costs money/tokens — the generation, the loop, the bulk write, the big fetch?
2. **Find the cheap gate.** What tiny check could make it unnecessary or smaller? Usually one of: an existence/dedup check, a metadata/topic compare, a query, a dry-run, or a small sample before the full run.
3. **Run the gate first.** Filter down to the minimal set that genuinely needs the expensive step.
4. **Prefer free + deterministic.** If plain code, an API, a query, a template, or a cached value gives the same result, use it instead of an LLM. Reserve model calls for what truly needs judgment.
5. **Do the expensive thing once, at the end, on the minimal set.** No generate-then-rework loops. Cache anything you'll reuse so you don't pay twice.

## Cheap vs. expensive — reach for the left first

| Instead of (expensive) | Try first (cheap) |
|---|---|
| Writing full articles then deduping | Compare topics/titles/clusters against existing content, then write survivors |
| LLM classify/extract in a loop | A query, regex, or map over structured data |
| Regenerating an image every run | Reuse the stored asset; only generate when missing |
| Re-fetching a whole dataset per item | Fetch once, cache, read from cache |
| Big model for a mechanical step | Cheapest model that still works (default Haiku, step up only with a reason) |
| Bulk-writing then fixing | Validate/diff against current state, write only what changed |

## Anti-pattern to catch

The **produce-then-discard** smell: any plan where the *first* action is to generate/write/fetch a lot, and a *later* step throws part of it away for a reason that was knowable up front (it already existed, it overlapped, it wasn't needed, it didn't fit). If you can see the discard coming, move that check to the front.

## Relationship to token-efficiency-check

- **This skill (`check-cost`)** is the everyday, in-task principle: sequence any piece of work cheap→expensive.
- **[[token-efficiency-check]]** is the one-time design gate you run right before building or scheduling an agent / task / cron / loop (model sizing, cadence, redundant runs, scope).

Use `check-cost` while doing the work; use `token-efficiency-check` when standing up a new automation.
