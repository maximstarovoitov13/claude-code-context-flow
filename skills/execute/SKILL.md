---
name: execute
description: Implement a plan file from context/plans/ step-by-step in a fresh session — the plan is the primary context. Use after /plan-feature and a /clear.
argument-hint: <path-to-plan-file (default: most recent PROPOSED in context/plans/)> [task numbers]
---

# Execute Plan

Implement the plan at: **$ARGUMENTS**

If no path given, find the most recent `PROPOSED` plan:

```bash
ls -t context/plans/*.md
```

This runs with the plan as the primary context — no planning-conversation
baggage. Trust the plan; if reality contradicts it, stop and flag, don't
improvise.

Read `context/PROJECT.md` for this project's gate commands and conventions.

## Process

### 1. Load the plan
Read it in full. Restate the goal and the task list in one or two lines so
the user can confirm you're working the right plan. If specific task numbers
were given, work only those.

### 2. Pre-flight
```bash
git status                    # clean? if not, ask before proceeding
git branch --show-current     # on main? branch first per PROJECT.md naming
```
Run any prerequisite regen steps from `PROJECT.md`'s drift guards if the plan
depends on fresh generated artifacts.

### 3. Work the tasks in order
For each task in the plan:
- Implement exactly what it specifies, following the conventions checklist in
  `PROJECT.md` and `CLAUDE.md`. Reuse existing helpers — grep before building.
- After each logical group, run the **fast checks** from `PROJECT.md` scoped
  to the touched area.
- If a task can't be done as written (missing dependency, wrong assumption),
  **stop and report** with what you found — don't silently deviate.

### 4. Gate
Run the full gate from `PROJECT.md`:
- **Slow checks in the BACKGROUND** — kick off, poll, never foreground-block.
- Fast checks and drift guards inline.
- Apply `PROJECT.md`'s "Gate interpretation" (e.g. zero-new-errors vs. fully
  green).

### 5. Audit-then-fix
After the feature works, spawn a reviewer sub-agent over the diff. Triage its
findings; ship genuine fixes as a **separate** commit before considering it
done.

### 6. Update the plan's status
If all tasks are done, mark the plan `Status: SHIPPED` and move it:
```bash
git mv context/plans/{name}.md context/plans/shipped/{name}.md
```
Update its row in `context/README.md` if that index exists. For partial
executions, tick the completed tasks in place instead.

### 7. Manual verification
Exercise the change per `PROJECT.md`'s "Manual verification" section — real
behavior, not just a green gate.

## Output
- Summary of what was implemented vs. what the plan called for (note any
  deviations).
- Real gate output (tests/typecheck/drift), not "should pass".
- Suggest `/commit`, then `/ship`.
