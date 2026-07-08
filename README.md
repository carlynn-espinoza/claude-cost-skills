# claude-cost-skills

Two small [Claude Code](https://claude.com/claude-code) skills that keep the cost of a task as close to $0 as possible **without losing the result**. They're a bundle because they solve the same problem at two different moments: one while you're *doing* work, one when you're *building automation*.

| Skill | When it fires | What it does |
|---|---|---|
| **check-cost** | Before any expensive step in a task | Run the cheap check that could make the expensive work unnecessary or smaller *before* you pay for it. Never generate-then-discard. |
| **token-efficiency-check** | Right before you build/schedule an agent, cron, or loop | A one-time design gate: right-size the model, kill redundant runs, cap scope, match cadence to how often data actually changes. |

## Why

Most wasted spend isn't the model being expensive — it's **doing expensive work you could have skipped**. The canonical example: writing three full blog articles, then throwing two away because they duplicated existing posts. Comparing the *topics* against existing content first (cheap) and only writing the survivors (expensive) reaches the same result for a fraction of the cost.

`check-cost` makes that ordering a habit. `token-efficiency-check` applies the same instinct to the automations you build, so a bad cadence or an oversized model doesn't quietly bill you every hour.

## Install

```
/plugin marketplace add https://github.com/carlynn-espinoza/claude-cost-skills
/plugin install cost-skills@claude-cost-skills
```

Then `/reload-plugins` (or restart the session). The skills auto-trigger from their descriptions — you don't have to call them by name, though you can.

> Type these in the Claude Code prompt (first character `/`), not in your terminal shell.

## What each skill actually checks

### check-cost — cheapest path first
1. **Name the expensive step** (generation, an LLM loop, a bulk write, a big fetch).
2. **Find the cheap gate** that could make it unnecessary or smaller — a dedup/overlap/existence check, a query, a dry-run, a sample.
3. **Run the gate first**, shrink to the minimal set that truly needs the expensive step.
4. **Prefer free + deterministic** — a query, plain code, a template, or a cached value over a model call.
5. **Do the expensive thing once, at the end, on what's left.** No generate-then-rework loops.

Catches the **produce-then-discard smell**: any plan whose first move is to generate/write/fetch a lot, and a later step throws part of it away for a reason that was knowable up front.

### token-efficiency-check — design-time gate
A short PASS/FLAG review across four axes before you build or schedule an automation:
1. **Redundant / duplicate runs** — does something already do this? Is there a dedup guard?
2. **Model right-sizing** — does this even need an LLM? If so, the cheapest one that works.
3. **Scope / context bloat** — pull only what's needed; cache reused input; cap output; cheap no-op exit.
4. **Cadence / frequency** — run only as often as the underlying data changes; idempotent and safe to re-run.

If everything passes, it says so in one line and gets out of the way. If something flags, it stops and hands you a concrete fix.

## License

MIT © Carlynn Espinoza
