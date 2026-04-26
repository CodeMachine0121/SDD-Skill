---
name: tdd
description: >
  Drive implementation via TDD. Reads a PRD, extracts test cases, then runs
  red-green-refactor per test case. Tests verify behavior only.
  Triggered by: "tdd", "implement with tdd", "/tdd".
---

## Pre-flight

1. Find `*-PRD.md` in `.sdd/`. None → stop ("Run `prd` first"). Multiple → ask user which one.
2. Find `.sdd/UL-MAP.md` (fallback: root, `docs/`). Missing → warn, continue.
3. Check SQL `test_cases` for existing rows matching this PRD → if found, ask to resume (skip to Cycle) or reset.

---

## Phase 1 — Test Plan

1. Parse PRD Section 3 ACs into atomic test cases (one behavior each). Skip empty/TBD rows. Also propose derived cases from Sections 4 & 6 edge cases/business rules.
2. SQL schema: `test_cases(id TEXT PK, prd_file TEXT, user_story TEXT, description TEXT, status TEXT DEFAULT 'pending')`
3. Present plan to user for confirmation (add/remove/reword). Apply edits to SQL, then start Cycle.

---

## Cycle — RED 🔴 → GREEN 🟢 → REFACTOR 🔵

For each `pending` test case:

**RED** — Write one failing test per [test-design.md](references/test-design.md).
- Run **this test only**. Must fail because behavior doesn't exist yet (not due to setup errors).

**GREEN** — Write minimum code to pass.
- Run **this test only**. Must pass before proceeding.

**REFACTOR** — Apply moves from [refactor.md](references/refactor.md) without changing behavior.
- Run **all module tests** after each change. All must stay green.
- Mark test case `done` in SQL.

Repeat until no `pending` rows remain. Mark unresolvable cases `blocked` with reason.

---

## Done

Run full suite. Report: "TDD complete. N done, M blocked. All tests passing."
