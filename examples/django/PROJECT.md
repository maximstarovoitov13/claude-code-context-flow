# Project config — read by context-flow skills

<!-- Example preset: Django + DRF + pytest. Adapted from a real production
     setup — replace paths/apps with your own or let /context-init detect them. -->

## Stack
Django 5 + DRF + pytest, virtualenv at `venv/`, OpenAPI schema via drf-spectacular.

## Gate (validation that must pass before shipping)

### Fast checks (run inline, after each logical group of changes)
```bash
venv/bin/pytest tests/test_{app}* -q                          # targeted tests for the touched app
venv/bin/python manage.py makemigrations --check --dry-run    # no unexpected migrations
```

### Slow checks (run in the BACKGROUND during /ship, poll — never foreground-block)
```bash
venv/bin/pytest tests/ -q --no-header -p no:warnings          # full suite, ~10 min
```

### Drift guards (regenerated artifacts that must be committed if dirty)
```bash
make schema && git status --short openapi.yml                 # OpenAPI regen — must be zero-warning
```

### Gate interpretation
Full suite must be 100% green. `makemigrations --check` failing means a model
change has no migration — hard stop.

## Branching & merging
- **Branch naming:** `<type>/<short-slug>`; never commit features to main
- **Merge strategy:** rebase — delete branch after merge
- **Post-merge (local):** `git pull --ff-only && venv/bin/python manage.py migrate`

## Commit conventions
- **Format:** conventional commits: `<tag>(<scope>): summary` — tags: feat fix refactor perf test docs chore
- **Scopes:** the Django app (`payments`, `reports`, `accounts`, …)
- **Trailer:** None
- **PR body footer:** `🤖 Generated with [Claude Code](https://claude.com/claude-code)`
- **Never stage:** `.env`, `settings.local.json`, `venv/`, `__pycache__/`, credentials, migrations unrelated to the task

## Cross-cutting surfaces (easy to break silently — checked by /commit and /ship)
- Permission group changes → permissions cache version bumped inline in the migration?
- Notification builders → used the frontend-routes helpers, not f-strung URLs?
- New email type → template file + per-template preview entry both exist?
- Serialized snapshot fields → added to the flattening allow-list?

## Conventions checklist (architecture rules /plan-feature and /execute enforce)
- Business logic lives in `services/`, not views — views are thin (parse → perm → service → serialize)
- Transactions at the service boundary: `with transaction.atomic():`; `select_for_update` inside it
- Audit + notifications fire from services, inside the same atomic block
- Permissions checked in views via the resolver; `404` not `403` for out-of-scope reads
- New state? Prefer deriving from existing tables over adding a model
- Migrations: UUID PKs, TextChoices, soft-delete where it touches invoicing/audit

## Manual verification
Hit the affected endpoint with the dev server (`venv/bin/python manage.py runserver`)
or the relevant management command; check audit rows and notification previews.

## Areas / modules
accounts / audit / notifications / companies / events / venues / exhibitors /
contractors / documents / billing / invoices / payments / service_orders /
checklists / reports / uploader / common
