---
name: ship
description: Take the current branch from "committed" to "merged into main, context updated" — pre-flight gate, context surfaces, push, PR, merge, land locally, delete stale HANDOFF.md. Use after /commit.
argument-hint: [PR number to resume merging, or empty to ship the current branch]
---

# Ship

Take the current branch from "committed" to "merged, landed locally, context
updated". This is the post-`/commit` half of the PR cadence.

Read `context/PROJECT.md` for the gate commands, merge strategy, and
post-merge steps. If `$ARGUMENTS` names an existing PR, skip to step 4
(pre-flight still applies if the working tree has unpushed commits).

## 0. Guard

```bash
git branch --show-current   # refuse if on the default branch — /ship merges INTO it
git status                  # working tree must be clean; if not → suggest /commit first
```

## 1. Pre-flight gate (all must pass BEFORE opening the PR)

From `PROJECT.md`:
- **Slow checks** (full suite, build): start in the **background** now, poll
  while doing the rest. Never foreground-block.
- **Fast checks**: run inline.
- **Drift guards**: run each regen command; if a generated artifact is dirty
  afterwards, **it belongs in this PR** — commit it (amend or a `chore`
  follow-up on the same branch). Stale generated artifacts are the classic
  silent miss this step exists to catch.
- Apply `PROJECT.md`'s "Gate interpretation" (fully-green vs. zero-new-vs-main).

Any hard failure → stop, fix on the branch, restart pre-flight.

## 2. Context surfaces

Verify each — skip only with a stated reason:

- [ ] **CLAUDE.md** updated for what shipped (module row, pattern to reuse,
      or anti-pattern learned)
- [ ] **Plan file** moved to `context/plans/shipped/` with `Status: SHIPPED`
      (+ its row in `context/README.md`), if this ships a planned feature —
      or tasks ticked for partial ships
- [ ] **Downstream docs** from the plan's "Downstream impact" section
      created/updated
- [ ] **Cross-cutting surfaces** from `PROJECT.md` — paired obligations all
      satisfied

If any of these produce changes, commit them (they ride in this PR).

## 3. Push + open PR

```bash
git push -u origin <branch>
gh pr create --title "<same style as the head commit>" --body "..."
```

PR body: summary bullets of WHAT + WHY, a **Test plan** section (how to verify
manually, per `PROJECT.md`), and the PR footer `PROJECT.md` requires, if any.

## 4. Wait on the gate + CI, then merge

Poll the background checks and `gh pr checks <num> --watch`. Merge only on a
clean pass, using `PROJECT.md`'s merge strategy:

```bash
gh pr merge <num> --{squash|rebase|merge} --delete-branch
```

- Anything fails → report the failures, fix on the branch, push, re-run. Do
  NOT merge red, even for "unrelated" flakes — name the flake explicitly and
  get the user's go-ahead first.

## 5. Land locally

```bash
git checkout <default-branch>
git pull --ff-only
```

Then run `PROJECT.md`'s post-merge steps (migrations, dependency install,
pruning the local branch after a squash-merge, …).

If a `HANDOFF.md` exists for the feature just merged, it's now stale — check
it for any Key Decision / Dead End not yet captured in the shipped plan or
`CLAUDE.md` (move it if so), then delete the file. A present `HANDOFF.md`
must always mean "in-flight work to resume".

## 6. Confirm

Report: PR number + URL, merge commit, gate results (real numbers), post-merge
steps run, and which context surfaces were touched. If anything was
deliberately skipped, say so.
