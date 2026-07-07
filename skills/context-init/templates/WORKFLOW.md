# Context & Session Management Workflow

How this repo is built with Claude Code: the artifacts, the skills, and the
session lifecycle that connects them. The core idea — **a session's context
window is a scratchpad, not a memory**. Anything worth keeping is externalized
into durable files; anything not worth keeping is deliberately thrown away by
starting fresh sessions. The model works best when its context contains only
what the current task needs.

---

## 1. The principle

Every Claude session has a context window. Two things degrade long sessions:

1. **Noise accumulation** — after hours of work, most of the context is stale
   exploration (files read while deciding, dead-end diffs, test output). It
   crowds out signal.
2. **Attention falloff** — precision on details from 200k tokens ago (exact
   field names, a constraint stated once) is softer than on recent context.

The fix is not bigger windows — it's **externalize + reset**. A fresh session
reading a 100-line curated artifact outperforms a 300k-token session, because
its context is 100% signal.

House rule: treat **~200–300k tokens as the soft cap**. When approaching it
mid-task, hand off at the next green checkpoint rather than pushing on.

---

## 2. The durable artifacts (externalized memory)

| Artifact | Role | Lifetime |
|---|---|---|
| `CLAUDE.md` | Living map of the system: conventions, gotchas, module pointers. Auto-loaded into **every** session — keep it terse, push detail into the docs below. | Permanent, edited every ship |
| `context/PROJECT.md` | Toolchain config: gate commands, merge cadence, cross-cutting surfaces. Read by every context-flow skill. | Permanent, updated when the toolchain changes |
| `context/plans/*.md` | Implementation plan for a feature (written by `/plan-feature`). The **source of truth** during execution — corrections go into the plan, not into chat. | Until shipped |
| `context/plans/shipped/*.md` | Archive of completed plans — the WHY behind each module. | Permanent |
| `HANDOFF.md` (repo root, gitignored) | Session-scoped baton for **in-flight, interrupted** work. Its *presence* is the signal: "there is unfinished work to resume". | Transient — deleted by `/ship` |
| Git history / PRs | The WHAT and WHY of every change (commit bodies explain WHY). | Permanent |

Rule of thumb for where a fact goes:
- Needed by *every* session → `CLAUDE.md` (one line + a pointer).
- Needed when working on *that module* → the plan or a module doc.
- Needed by *the next session only* → `HANDOFF.md`.
- Needed *never again* → nowhere; let `/clear` discard it.

---

## 3. The skills (one per lifecycle step)

| Skill | What it does |
|---|---|
| `/plan-feature` | Research the codebase with sub-agents, write a task-numbered plan to `context/plans/`. Heavy exploration happens **here**, in a session that will be thrown away. |
| `/execute <plan> [tasks]` | Implement the plan in a **fresh** session — plan as primary context, zero planning-conversation baggage. Trusts the plan; flags contradictions instead of improvising. |
| `/commit` | Atomic commit: conventional format, WHY-focused body, cross-cutting-surface checklist from `PROJECT.md`. Never pushes. |
| `/ship` | The post-commit ritual: pre-flight gate (slow checks in the background), context surfaces updated, push → PR → poll checks → merge → land locally → delete stale `HANDOFF.md`. |
| `/handoff` | Write `HANDOFF.md`: goal, done, next steps, **key decisions + dead ends**, exact first action. For interrupted work only. |

---

## 4. The lifecycle

### Planned feature (the main loop)

A feature (e.g. 15 tasks) is split into phases; each phase = one branch = one
reviewable PR. Ship per **main-safe slice**, not per finished feature.

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

**No `/handoff` between cleanly-shipped phases.** Everything Phase N+1 needs is
already durable (plan with tasks marked done, merged code, CLAUDE.md). If a
phase learned something the plan didn't anticipate, **edit the plan file** —
that's what `/execute` reads.

### Interrupted work (the exception path)

Phase half-done, context near the cap, or end of day:

```
/commit (on the branch — no /ship, the branch waits)
/handoff  ──►  HANDOFF.md
/clear
...next session: "Read HANDOFF.md and continue"
```

Also used for unplanned work with no plan file to carry state (bug hunts,
exploration that turned into building).

### Quick questions / advice

Just `/clear` and ask. One concern per session — a design question doesn't
belong in a feature session's context, and vice versa.

---

## 5. Decision rules (the whole thing in four questions)

1. **Session end: "Is the remaining work fully described by a durable
   artifact?"** Yes → just `/clear`. No → `/handoff` first.
2. **Slice done: "Can this merge alone — main green, coherent to review,
   nothing misleading half-wired?"** Yes → `/ship` now; don't wait for the
   whole feature. No → keep the branch, `/handoff` if stopping.
3. **When to hand off: at ~20–30% context remaining or the next green
   checkpoint after ~200–300k tokens** — while the session is still sharp
   enough to write a good handoff. Never ride into auto-compaction: the
   auto-summary doesn't know which decisions and dead ends matter.
4. **Where does this fact live?** (see table in §2). If tempted to write a
   handoff that reads like a plan, the work deserved `/plan-feature`.

---

## 6. HANDOFF.md lifecycle

```
/handoff creates it ──► next session consumes it
        ▲                    │
        └── overwritten if ──┘
            still unfinished
                             ▼
              /ship deletes it after merge
              (salvaging any Key Decision / Dead End
               into the plan or CLAUDE.md first)
```

Delete, never leave stale: a present `HANDOFF.md` must always mean "in-flight
work". An absent file can't mislead a cold session; a stale one can.

---

## 7. Why this works

- **Plan/execute split** = the expensive exploration happens in a disposable
  session; execution starts with a clean, high-signal context.
- **Per-slice shipping** = smaller PRs, main always green, failures bisect to
  less code, `pull --ff-only` never fights conflicts.
- **`/ship`'s pre-flight** = the historically-missed steps (regenerated
  artifacts, migration checks, doc updates) are structural, not
  memory-dependent.
- **Curated handoffs over auto-compaction** = *you* choose what survives a
  session boundary, and dead ends are preserved so they're never re-walked.
- **Bus factor** = every decision trail (plans, PR bodies, docs) is readable
  by a human teammate, not locked in chat history.
