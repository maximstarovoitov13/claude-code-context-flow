# Project config — read by context-flow skills

<!-- Written by /context-init. Every command here must be real and runnable
     in this repo. Update this file when the toolchain changes. -->

## Stack
{e.g. Django 5 + DRF + pytest · Next.js 15 + TypeScript + SWR · Go 1.23 …}

## Gate (validation that must pass before shipping)

### Fast checks (run inline, after each logical group of changes)
```bash
{e.g. venv/bin/pytest tests/test_{area}* -q        # targeted tests}
{e.g. npx tsc --noEmit                             # typecheck}
```

### Slow checks (run in the BACKGROUND during /ship, poll — never foreground-block)
```bash
{e.g. venv/bin/pytest tests/ -q --no-header        # full suite, ~N min}
{e.g. npm run build}
```

### Drift guards (regenerated artifacts that must be committed if dirty)
```bash
{e.g. make schema && git status --short openapi.yml}
{e.g. npm run gen:api && git status --short src/types/api.d.ts}
```
<!-- "None" if the repo has no generated artifacts -->

### Gate interpretation
{e.g. "Whole-project tsc carries pre-existing errors on main — the bar is zero
NEW errors in touched files, not green." Or: "Full suite must be 100% green."}

## Branching & merging
- **Branch naming:** {e.g. `<type>/<short-slug>`; never commit features to main}
- **Merge strategy:** {squash | rebase | merge} — {delete branch after merge?}
- **Post-merge (local):** {e.g. `git pull --ff-only && venv/bin/python manage.py migrate` · `npm install` if lockfile moved · "None"}

## Commit conventions
- **Format:** {e.g. conventional commits: `<tag>(<scope>): summary` — tags: feat fix refactor perf test docs chore}
- **Scopes:** {e.g. the Django app · the feature area}
- **Trailer:** {required Co-Authored-By / sign-off line, or "None"}
- **PR body footer:** {required footer line, or "None"}
- **Never stage:** {e.g. `.env`, `venv/`, `node_modules/`, credentials, unrelated generated files}

## Cross-cutting surfaces (easy to break silently — checked by /commit and /ship)
<!-- Things a change can touch that need a paired action elsewhere. -->
- {e.g. Model change → migration created? Permission change → cache version bumped?}
- {e.g. New endpoint → cache-bust prefixes updated? New email type → template + preview entry?}
- "None known yet" is a valid start — add entries as they bite you.

## Conventions checklist (architecture rules /plan-feature and /execute enforce)
- {e.g. Business logic in services/, views stay thin}
- {e.g. Types come from generated api.d.ts — never hand-typed}
- {e.g. 404-not-403 for out-of-scope reads}

## Manual verification
{How to exercise a change by hand: URLs/ports/subdomains, seed commands, test accounts.}

## Areas / modules
{The vocabulary of this codebase — app names, feature areas, portals — so plans
scope themselves correctly. e.g. `accounts / billing / events / reports / …`}
