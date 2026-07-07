---
name: commit
description: Create a focused, atomic commit following this project's conventions (from context/PROJECT.md) — branch guard, explicit staging, WHY-focused body, cross-cutting-surface checklist. Never pushes.
---

# Commit

Create a focused, atomic commit for the current changes.

Read `context/PROJECT.md` for this project's commit format, never-stage list,
and cross-cutting surfaces. If it doesn't exist, suggest `/context-init` and
fall back to conventional commits + common sense.

## 1. Review

```bash
git status
git diff HEAD
git ls-files --others --exclude-standard   # untracked
```

## 2. Branch guard

If on the default branch and `PROJECT.md` says features never land there
directly, branch first using the project's naming convention:

```bash
git checkout -b <type>/<short-slug>
```

Skip only if the user explicitly said to commit on the current branch.

## 3. Stage

Stage only files relevant to *this* change, **by explicit path**. Never stage
anything on `PROJECT.md`'s never-stage list — and regardless of config, never
stage credentials, `.env` files, local settings, dependency directories, or
build artifacts. Confirm nothing unrelated slipped in.

## 4. Message

Follow `PROJECT.md`'s commit format and scope vocabulary, matching the style
visible in `git log --oneline -10`.

Body explains **WHY**, not what (the diff shows what). Keep it tight.

## 5. Flag cross-cutting surfaces in the body

Walk `PROJECT.md`'s "Cross-cutting surfaces" list. If the change touches any,
note in the body how the paired obligation was handled — so the log stays a
usable memory. Also: if the change touched AI context files (`CLAUDE.md`,
`.claude/`, `context/`), add a `Context:` line listing what changed.

## 6. Trailer

Append the trailer required by `PROJECT.md`, if any.

## 7. Confirm

Print the commit hash + one-line summary. Do **not** push unless asked —
push/PR/merge is `/ship`'s job.
