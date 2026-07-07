# Project config — read by context-flow skills

<!-- Example preset: Next.js + TypeScript + SWR with generated API types.
     Adapted from a real production setup — replace with your own or let
     /context-init detect it. -->

## Stack
Next.js 15 (App Router) + TypeScript + MUI + SWR; API types generated from an
OpenAPI schema (`npm run gen:api` → `src/types/api.d.ts`). No test runner —
the gate is build + tsc + lint + format.

## Gate (validation that must pass before shipping)

### Fast checks (run inline, after each logical group of changes)
```bash
npx tsc --noEmit                                              # whole-project typecheck
FILES=$(git diff main...HEAD --name-only -- 'src/**/*.ts' 'src/**/*.tsx')
npx eslint $FILES
npx prettier --check $FILES
```

### Slow checks (run in the BACKGROUND during /ship, poll — never foreground-block)
```bash
npm run build                    # catches SSR/import/route errors tsc misses
```

### Drift guards (regenerated artifacts that must be committed if dirty)
```bash
npm run gen:api && git status --short src/types/api.d.ts
```

### Gate interpretation
Main carries pre-existing tsc/lint debt — the bar is **zero NEW errors**, none
in files this branch touched (compare error count to main's baseline). Build
verdict is "no new failure vs main"; confirm a failing file is untouched by
the branch before treating it as pre-existing.

## Branching & merging
- **Branch naming:** `<type>/<short-slug>`; never commit features to main
- **Merge strategy:** squash — delete branch after merge (and `git branch -D` locally, squash won't auto-prune)
- **Post-merge (local):** `git pull --ff-only`; `npm install` if `package-lock.json` moved

## Commit conventions
- **Format:** conventional commits: `<tag>(<scope>): summary` — tags: feat fix refactor perf test docs chore
- **Scopes:** the feature area (`invoices`, `reports`, `floor-plan`, `api`, …)
- **Trailer:** None
- **PR body footer:** `🤖 Generated with [Claude Code](https://claude.com/claude-code)`
- **Never stage:** `.env`, `.claude/settings.local.json`, `node_modules/`, `.next/`, `*.pem` dev certs, designer assets you didn't intentionally change

## Cross-cutting surfaces (easy to break silently — checked by /commit and /ship)
- Generated API types (`src/types/api.d.ts`) → regenerated via `npm run gen:api`, never hand-edited
- New endpoint hooks → cache-bust prefix keys added to the `bust*Caches` helper?
- Shared portal views → `routes` prop kept so links stay in the right portal?
- Dark mode → `theme.vars.palette.*` (not `theme.palette.*`) in `sx`?
- Permission-gated UI → `usePermission` / `useEventPermission` wrapped?

## Conventions checklist (architecture rules /plan-feature and /execute enforce)
- Page files are thin — they import a view from `src/sections/.../views/`
- Forms = React Hook Form + zod (`zodResolver`); submit via an action function, never axios in section files
- Data = SWR hooks in `src/actions/<entity>.ts` with a cache-bust helper; string URL keys
- Types come from generated `api.d.ts` — never hand-type API shapes
- Endpoints in `src/lib/axios.ts`; app URLs in `src/routes/paths.ts`
- Money is a string; lists use the pagination envelope (`.results`); 404-not-403

## Manual verification
Exercise on the right surface: `admin.localhost:8082` (staff),
`<slug>.localhost:8082` (event portal), `localhost:8082` (apex). Confirm
pagination renders, cookies flow, dark mode holds.

## Areas / modules
<!-- Your feature-area vocabulary, so plans and commit scopes name things
     consistently. e.g.: -->
auth / billing / reports / notifications / settings / users
