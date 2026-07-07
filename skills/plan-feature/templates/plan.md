# Plan: {Feature Name}

Status: PROPOSED

## Overview
{1-2 sentences: what + why.}

## Success Criteria
- [ ] {Verifiable criterion}
- [ ] Gate green: {the fast + slow check commands from context/PROJECT.md}

## Affected Areas
- `{area}` — {what changes}

## Architecture Notes
{Key decisions, tradeoffs, which existing helpers/primitives are reused,
interface changes.}

## Implementation Tasks

### Task 1: {name}
**Files:** `{path}`, …
**Type:** Create | Modify | Delete
**Description:** {what + why}
**Depends on:** {Task N | none}

### Task 2: …

## Validation
1. Targeted: {targeted test/check command for the touched area}
2. Full gate (background if slow): {slow check commands}
3. Drift guards: {regen commands + "no diff" expectation, or "None"}
4. Manual: {specific URL/command/flow to exercise, per PROJECT.md}

## Downstream impact
{Changed contracts/endpoints/types other code or repos depend on → which docs
or consumers need updating. "None" if self-contained.}

## Rollback
{How to revert safely.}
