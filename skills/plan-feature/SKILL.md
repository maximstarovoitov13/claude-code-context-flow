---
name: plan-feature
description: Research the codebase with sub-agents, then write an implementation plan file to context/plans/ for /execute to consume in a fresh session. Use when starting any non-trivial feature or change.
argument-hint: <feature-name-or-description>
---

# Plan Feature

Produce a detailed, actionable implementation plan for: **$ARGUMENTS**

The plan is saved to `context/plans/{kebab-case-name}.md` and is designed to
be consumed by `/execute` in a fresh session. Planning happens here; building
happens there — keep the two separate so execution starts with a clean context.

First, read `context/PROJECT.md` for this project's gate commands, conventions
checklist, and area vocabulary. If it doesn't exist, suggest `/context-init`
and fall back to what `CLAUDE.md` and the repo itself reveal.

---

## Phase 1: Understand the request

Restate it in your own words. Identify:

1. **Problem** — what gap or pain does this close?
2. **Success criteria** — what does "done" look like, and how do we verify it?
3. **Scope** — explicitly in scope vs. out of scope.
4. **Affected areas** — which modules/apps/surfaces from `PROJECT.md`'s
   "Areas" section does this touch?

## Phase 2: Codebase intelligence (ISOLATE — use sub-agents)

Spawn parallel **Explore** sub-agents so research noise stays out of this
session. Each returns a short findings summary, not file dumps:

- **A — Area deep-dive:** map the current code in the affected area(s) —
  models/components, business logic, entry points. List every file that will
  change.
- **B — Reuse scan:** grep for existing helpers, shared primitives, and
  patterns before proposing anything new. Check `PROJECT.md`'s conventions
  checklist and `CLAUDE.md` for "patterns to reuse". Mature codebases have
  most of what a feature needs already wired.
- **C — Test & contract patterns:** read 2-3 representative tests, factories,
  fixtures; if the project has generated API types or contract docs, read the
  relevant ones and note exact shapes.
- **D — Prior art:** `git log --oneline -20 -- <area>/`; skim any matching
  doc in `context/plans/` or `context/plans/shipped/`.

Synthesize: current state, gaps, constraints.

## Phase 3: External research (only if needed)

If the feature touches an external API/SDK/unfamiliar library, use web search
for docs + known gotchas. Skip otherwise.

## Phase 4: Strategic thinking

Reason through the design **against this repo's conventions** — the
"Conventions checklist" in `context/PROJECT.md` and the rules in `CLAUDE.md`.
Additionally, always consider:

- **Reuse over new:** can this derive from existing code/data instead of
  adding a new structure?
- **Blast radius / rollback:** is the change reversible without destructive
  steps (e.g. data-destroying migrations)? Does it touch a cross-cutting
  surface listed in `PROJECT.md`?

## Phase 5: Write the plan

Save to `context/plans/{kebab-case-name}.md` using this skill's
`templates/plan.md`. Fill every section; the Validation section must use the
**real gate commands from `PROJECT.md`**, not generic placeholders.

### Ordering rules
- Foundations first (schema/types/contracts) → core logic → UI/entry points
  → tests → docs.
- Group tasks by area to minimize context switching.
- Blocked tasks after their dependencies.

## Output

1. Save the plan file (print its full path) and add a row for it in
   `context/README.md` if that index exists.
2. Print the plan to the conversation.
3. Summarize: task count, affected areas, complexity (low/med/high), open
   questions to resolve **before** `/execute`.
4. Remind: `/clear`, then `/execute context/plans/{name}.md` in a fresh
   session.
