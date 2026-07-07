---
name: handoff
description: Externalize this session's working memory into HANDOFF.md so the next session picks up cold — goal, done, next steps, key decisions, dead ends, exact first action. Use before ending a session with in-flight work, or when context gets tight.
---

# Handoff

Externalize this session's working memory into a file the next session reads
to pick up cold. Write + compress: capture the essentials, link to files
rather than pasting them.

## When
- Before ending a long session where work continues later.
- Proactively, before context gets tight — not after. Never ride into
  auto-compaction.
- When switching phases (research → implementation).
- **Not** between cleanly-shipped phases of a planned feature — the plan file
  is the handoff there; update the plan instead.

## 1. Gather state
```bash
git status
git diff --stat HEAD
git log --oneline -5
git branch --show-current
```

## 2. Write `HANDOFF.md` (repo root)

Use this skill's `templates/handoff.md`. The "Current State" section should
reflect this project's gate (see `context/PROJECT.md`): which checks pass,
which artifacts are stale, what was manually verified.

## 3. Confirm
- Print the file's full path.
- If there are uncommitted changes, suggest `/commit` first.
- Next session starts with: *"Read HANDOFF.md and continue."*

## Quality bar
- Under ~100 lines. A fresh agent should continue without asking questions.
- Keep "Key Decisions" and "Dead Ends" — they stop the next session reversing
  choices or re-walking failed paths.
- Reference paths; never paste file contents or debug transcripts.

## Note vs. durable context
`HANDOFF.md` is **session-scoped** and transient (overwritten next session,
deleted by `/ship`). Durable cross-session facts go in `CLAUDE.md` or your
memory system; shipped-feature rationale goes in `context/plans/shipped/`.
Don't duplicate those here.
