# context-flow

**A context-management lifecycle for Claude Code.** Five skills — `/plan-feature → /execute → /commit → /ship → /handoff` — plus an init command that adapts them to any stack.

The core idea: **a session's context window is a scratchpad, not a memory.** Anything worth keeping is externalized into durable files; anything not worth keeping is deliberately thrown away by starting fresh sessions. The model works best when its context contains only what the current task needs.

Battle-tested on a production Django API and its Next.js frontend; the project-specific parts (test commands, merge cadence, conventions) live in one config file, so the same skills work on Go, Rust, Rails, or anything else.

## Why

Two things degrade long Claude Code sessions:

1. **Noise accumulation** — after hours of work, most of the context is stale exploration: files read while deciding, dead-end diffs, test output. It crowds out signal.
2. **Attention falloff** — precision on details from 200k tokens ago is softer than on recent context.

The fix is not bigger windows — it's **externalize + reset**. A fresh session reading a 100-line curated plan outperforms a 300k-token session, because its context is 100% signal.

## Install

```
/plugin marketplace add maximstarovoitov13/context-flow
/plugin install context-flow@context-flow-marketplace
```

Then, in each project:

```
/context-flow:context-init
```

This detects your stack (Django, Next.js, Go, Rust, Rails, …), asks a few questions it can't detect (merge strategy, cross-cutting surfaces), and writes `context/PROJECT.md` — the config every other skill reads — plus the `context/` scaffolding and a `WORKFLOW.md` explaining the system to teammates.

## The lifecycle

```
/plan-feature  ──►  context/plans/feature.md          (throw-away research session)
     │ /clear
     ▼
/execute plan (Phase 1 tasks)                          (fresh session)
     │
/commit  ──►  /ship  ──►  merged, plan updated, main green
     │ /clear
     ▼
/execute plan (Phase 2 tasks)                          (fresh session — reads the
     │                                                  plan, NOT a handoff: the
    ...                                                 plan IS the handoff for
     ▼                                                  planned work)
final /ship  ──►  plan → context/plans/shipped/, CLAUDE.md finalized
```

| Skill | What it does |
|---|---|
| `/context-init` | One-time setup: detect the stack, write `context/PROJECT.md`, scaffold `context/`. |
| `/plan-feature <idea>` | Research the codebase with parallel sub-agents (noise stays out of your session), write a task-numbered plan to `context/plans/`. The expensive exploration happens in a session you'll throw away. |
| `/execute [plan]` | Implement the plan in a **fresh** session — plan as primary context, zero planning baggage. Runs your gate after each group; audit-then-fix before done. |
| `/commit` | Atomic commit: your conventions, WHY-focused body, cross-cutting-surface checklist. Never pushes. |
| `/ship` | Pre-flight gate (slow checks in the background), drift guards for generated artifacts, context surfaces updated, push → PR → merge → land locally. |
| `/handoff` | For **interrupted** work only: write `HANDOFF.md` with goal, next steps, key decisions, and dead ends, so the next session picks up cold. Deleted by `/ship`. |

## Where every fact lives

| Fact is needed… | It goes in… |
|---|---|
| by every session | `CLAUDE.md` (one line + a pointer) |
| when working on that module | the plan file / a module doc |
| by the next session only | `HANDOFF.md` (transient, gitignored) |
| never again | nowhere — let `/clear` discard it |

## The four decision rules

1. **Session end:** is the remaining work fully described by a durable artifact? Yes → just `/clear`. No → `/handoff` first.
2. **Slice done:** can this merge alone — main green, coherent to review? Yes → `/ship` now; don't wait for the whole feature.
3. **When to hand off:** at ~20–30% context remaining, at a green checkpoint — while the session is still sharp. Never ride into auto-compaction; the auto-summary doesn't know which decisions and dead ends matter.
4. **Where does this fact live?** (table above). If your handoff reads like a plan, the work deserved `/plan-feature`.

## Adapting to your stack

Everything project-specific is *data* in `context/PROJECT.md`; the skills are pure *procedure*. See two real filled-in examples:

- [`examples/django/PROJECT.md`](examples/django/PROJECT.md) — pytest suite in the background, migration checks, OpenAPI drift guard, rebase-merge
- [`examples/nextjs/PROJECT.md`](examples/nextjs/PROJECT.md) — no test runner (tsc/lint/build gate), generated-API-types drift guard, zero-new-errors-vs-main interpretation, squash-merge

## License

MIT
