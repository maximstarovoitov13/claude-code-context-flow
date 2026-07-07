---
name: context-init
description: Bootstrap context-flow in a project — detect the stack, interview the user, scaffold the context/ directory, and write context/PROJECT.md (the config every other context-flow skill reads). Run once per repo, or re-run to update the config.
---

# Context Init

Set up the context-flow system in this repository. Every other skill
(`/plan-feature`, `/execute`, `/commit`, `/ship`, `/handoff`) reads
`context/PROJECT.md` for project-specific commands and conventions —
this skill creates it.

If `context/PROJECT.md` already exists, read it and switch to **update mode**:
show the current config, ask what changed, and edit in place. Never blow away
an existing `context/` directory.

## 1. Detect the stack

Inspect the repo (don't ask what you can detect):

| Signal | Stack | Default gate |
|---|---|---|
| `manage.py` / `pyproject.toml` + django | Django | `pytest -q` (or `manage.py test`), `manage.py makemigrations --check --dry-run` |
| `pyproject.toml` / `setup.py` (no Django) | Python | `pytest -q` |
| `package.json` with `next` | Next.js | `npx tsc --noEmit`, `eslint` + `prettier --check` on touched files, `npm run build` |
| `package.json` (other) | Node/TS | `npx tsc --noEmit` (if TS), `npm test` if a test script exists |
| `go.mod` | Go | `go test ./...`, `go vet ./...` |
| `Cargo.toml` | Rust | `cargo test`, `cargo clippy` |
| `Gemfile` with rails | Rails | `bin/rails test` or `rspec`, `rails db:migrate:status` |
| other | generic | ask the user |

Also detect:
- **Test runner specifics**: virtualenv path (`venv/`, `.venv/`), package manager (npm/yarn/pnpm), Makefile targets.
- **Schema/codegen artifacts** that must be regenerated and committed (OpenAPI specs, generated API types, GraphQL schemas, lockfiles). Grep package.json scripts and Makefile for `gen`, `schema`, `openapi`.
- **CI**: `.github/workflows/` — note which checks gate PRs.
- **Approximate full-suite duration** if discoverable; anything over ~2 min should run in the background during `/ship`.

## 2. Interview the user

Ask only what can't be detected (use one round of questions):

1. **Merge strategy** — squash / rebase / merge commit? Delete branches after merge?
2. **Branch naming** — e.g. `<type>/<short-slug>`? Is committing to main ever OK?
3. **Cross-cutting surfaces** — "what's easy to break silently in this repo?" (caches to bust, permission systems, generated files, templates that must exist in pairs). These become the `/commit` checklist.
4. **Post-merge steps** — migrations to run, dependencies to install?
5. **Commit trailer / PR footer** — any required attribution lines?

## 3. Scaffold

Create (only what doesn't exist):

```
context/
├── PROJECT.md          # the config (template below)
├── README.md           # index of context/ — starts minimal, grows with plans
├── WORKFLOW.md         # copy from this skill's templates/WORKFLOW.md
└── plans/
    └── shipped/
```

Add `HANDOFF.md` to `.gitignore` (it is session-scoped and local).

If a `CLAUDE.md` exists, append a short section pointing at `context/PROJECT.md`
and `context/WORKFLOW.md`; if none exists, suggest running `/init` first.

## 4. Write `context/PROJECT.md`

Use `templates/PROJECT.md` from this skill, filled in with everything detected
and answered. Every field must contain a **real, runnable command** for this
repo — no placeholders left behind. If a section genuinely doesn't apply
(e.g. no codegen), write "None" so downstream skills don't guess.

## 5. Confirm

Print: the detected stack, the gate commands (fast vs. slow), the merge
cadence, and the created files. Then suggest the lifecycle:

> `/plan-feature <idea>` → `/clear` → `/execute` → `/commit` → `/ship`,
> with `/handoff` when stopping mid-work. See `context/WORKFLOW.md`.
