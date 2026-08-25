---
name: tdd
description: >
  Drive implementation via TDD against a feature's PRD.md (acceptance criteria)
  and ARCH.md (the technical design). Extracts test cases from the PRD scenarios,
  runs red-green-refactor per case, and commits every cycle. Tests verify
  behavior only.
  Triggered by: "tdd", "implement with tdd", "/tdd".
---

## Pre-flight

The design contract for this skill is exactly two documents: **`PRD.md`** (what to build + acceptance criteria) and **`ARCH.md`** (how it is structured). Read them and nothing else drives the implementation.

1. Locate the feature folder `.sdd/{yyyy-MM-dd}-{feature-slug}/`. None → stop ("Run `/prd` first"). Multiple candidates → ask which feature. Then read:
   - **`PRD.md`** — the Section 3 acceptance-criteria scenarios (Gherkin) are the behavior contract.
   - **`ARCH.md`** — the change scope and the new/modified classes and modules with their responsibilities. Implement to this design; do not invent a different structure. Missing → warn ("Run `/architecture` first for a design to follow") and continue only if the user insists.
2. Find `.sdd/UL-MAP.md` (fallback: root, `docs/`). Missing → warn, continue. Use its vocabulary for names.
3. **Load the [`commit`](../commit/SKILL.md) skill** — it owns the commit convention for this workflow (English Conventional Commits, one commit per cycle, no SDD identifiers). Follow it for every commit made in this run.
4. Check SQL `test_cases` for existing rows matching this PRD → if found, ask to resume (skip to Cycle) or reset.

---

## Phase 1 — Test Plan

1. Parse the PRD Section 3 acceptance-criteria scenarios (Gherkin) into atomic test cases — one behavior per scenario, preserving the happy / boundary / exception coverage. Skip empty/TBD rows. Also propose derived cases from Sections 4 & 6 edge cases/business rules. Map each case to the component in `ARCH.md` that owns it, so each test lands at the right boundary.
2. SQL schema: `test_cases(id TEXT PK, prd_file TEXT, user_story TEXT, description TEXT, status TEXT DEFAULT 'pending')` — the `id` is for internal tracking only and **never** appears in a commit message.
3. Present plan to user for confirmation (add/remove/reword). Apply edits to SQL, then start Cycle.

---

## Cycle — RED 🔴 → GREEN 🟢 → REFACTOR 🔵 → COMMIT ✅

For each `pending` test case:

**RED** — Write one failing test per [test-design.md](references/test-design.md).
- Run **this test only**. Must fail because behavior doesn't exist yet (not due to setup errors).

**GREEN** — Write minimum code to pass.
- Run **this test only**. Must pass before proceeding.

**REFACTOR** — Apply moves from [refactor.md](references/refactor.md) without changing behavior.
- Run **all module tests** after each change. All must stay green.
- Mark test case `done` in SQL.

**COMMIT** — Commit this cycle before starting the next test case, per the [`commit`](../commit/SKILL.md) skill.
- **One commit per completed TDD cycle** — the test and its implementation together. An **app integration** (wiring a component into the composition root / DI / an endpoint / config) gets its own commit, separate from behavior cycles.
- Stage only the files for this cycle, not unrelated changes.
- English Conventional Commits, describing the behavior — never a test-case id or document number.
- Every commit must be green and self-contained.

Repeat until no `pending` rows remain. Mark unresolvable cases `blocked` with reason.

---

## Done

Run full suite. Report: "TDD complete. N done, M blocked. All tests passing."
