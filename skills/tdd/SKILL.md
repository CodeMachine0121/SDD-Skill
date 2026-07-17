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
3. **Detect the project's commit style** — see [Commit Convention](#commit-convention) — so every cycle commit matches it from the first commit onward.
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

**COMMIT** — Commit this cycle before starting the next test case.
- Stage only the files for this cycle (the test + its implementation), not unrelated changes.
- Write the message per [Commit Convention](#commit-convention) — describe the behavior, never a test-case id or document number.
- **One commit per completed TDD cycle.** Additionally, commit an **app integration** on its own (wiring a component into the composition root / DI / an endpoint / config) so it is separate from behavior cycles.
- Every commit must be green and self-contained.

Repeat until no `pending` rows remain. Mark unresolvable cases `blocked` with reason.

---

## Commit Convention

Determine the style once in Pre-flight and reuse it for every commit:

1. Inspect recent history — `git log --oneline -20`. Match the project's existing convention: prefix style (`feat:`, `fix:`, …), language, casing, and whether a scope is used.
2. **If the project has no history (brand-new project)** → default to **English**, following **Conventional Commits**: `type(scope): summary` (e.g. `feat(cart): add item to cart`).

Rules that always hold, regardless of detected style:

- **No SDD identifiers in the message** — never include a test-case id, scenario/AC number, PRD/ARCH filename, or the feature-folder date. Commits describe *behavior*, not our internal document numbering.
- Subject line is imperative and concise; the test and implementation of one cycle go in the **same** commit.

---

## Done

Run full suite. Report: "TDD complete. N done, M blocked. All tests passing."
