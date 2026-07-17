---
name: contract
description: >
  Verify an agent's implementation conforms to the contract — the feature's
  PRD.md (preferred; its Gherkin acceptance-criteria scenarios) or BRIEF.md — via
  a Traceability Matrix. Map every contract clause to code + test, using ARCH.md's
  component map when present, run the suite, then flag gaps (unimplemented clauses)
  and orphans (code with no clause). Never edits source code.
  Triggered by: "contract", "trace contract", "verify contract", "/contract".
---

## What this verifies

Conformance by **executed coverage**. The matrix proves every contract clause is
*implemented and verified by a passing test*, and surfaces code that answers to
*no* clause. The full unit-test suite is run **once** so each clause's status
reflects real pass/fail, not just test existence.

It still does **not** prove the test itself correctly captures the clause's
intent (a passing test can assert the wrong thing). Report this residual
limitation in the summary; pair with `/tdd` for deeper behavioral confidence.

---

## Pre-flight

1. Find the contract source — the feature's spec document under `.sdd/{date}-{feature}/`:
   - **`PRD.md` is the contract** when present (richer: Gherkin acceptance criteria and business rules). Prefer it.
   - Else fall back to **`BRIEF.md`** (Requirements + Examples + Out of Scope).
   - Neither found → stop: "No contract found. Run `/prd` (or `/clarify`) first."
   - Multiple feature folders → ask user which one.
2. Read **`ARCH.md`** in the same folder if present. Its Traceability table (PRD scenario → fulfilling component) is a map, not a contract: use it in Phase 3 to jump to the code that should satisfy each clause. A scenario with no ARCH mapping, or a mapped component that does not exist in the code, is a strong gap signal.
3. Find `.sdd/UL-MAP.md` (fallback: root, `docs/`). Missing → warn, continue (clause/code matching will be weaker without the glossary).
4. Confirm the implementation location with the user if it is not obvious from the repo layout.

---

## Phase 1 — Extract contract clauses

Parse the contract into **atomic clauses**, one verifiable statement each. Assign stable IDs by source:

**When the contract is `PRD.md`:**

| Source | ID prefix |
|---|---|
| Section 3 — Acceptance Criteria (one per Gherkin `Scenario`) | `AC-` |
| Section 4 — Core Business Rules | `BR-` |
| Section 6 — Non-Functional Requirements | `NFR-` |

Each Gherkin `Scenario` is one atomic clause; its `Given/When/Then` is the verifiable statement. Keep the scenario title in the clause text so it traces back.

**When the contract is `BRIEF.md`:**

| Source | ID prefix |
|---|---|
| Requirements (per bullet) | `REQ-` |
| Examples (per example row) | `EX-` |

Skip empty/TBD rows. Keep the clause text verbatim so it is traceable back to the source.

Also capture the contract's **Out of Scope** items (PRD/BRIEF) as a negative checklist — used in Phase 3 to judge orphans: code implementing an out-of-scope item is a contract **violation**, not a benign orphan.

Present the extracted clause list to the user for confirmation before tracing (add / remove / reword).

---

## Phase 2 — Run the full test suite

Run the project's **entire** unit-test suite once (detect the runner from the repo — `npm test`, `pytest`, `dotnet test`, `go test ./...`, etc.; ask the user if ambiguous). This is the only command this skill executes; it mutates nothing.

- Capture per-test pass/fail results — this is what makes each clause's status reflect real behavior, not just test existence.
- If the suite fails to run at all (compile/setup error), stop and report — the matrix would be meaningless. Surface the error to the user.

---

## Phase 3 — Trace each clause

For every clause, search the implementation and the test suite (read-only) to answer two questions, then cross-reference the Phase 2 results:

- **Implementation** — which code satisfies it? Record `file:line` (the most specific site). If `ARCH.md` mapped this clause to a component, start there; otherwise use UL-MAP terms to bridge business wording → code names. A clause whose ARCH-mapped component is absent from the code is a `gap`.
- **Verification** — which test asserts it? Record the test name / `file:line`, and whether it **passed or failed** in the Phase 2 run.

Then classify status:

| Status | Meaning |
|---|---|
| ✅ `covered` | has implementation **and** a test that **passed** |
| 🔴 `failing` | clause has a test but it **failed** — implementation violates the contract |
| 🟡 `partial` | implemented but no test (or test exists, implementation unclear) |
| ❌ `gap` | no implementation found |
| ⚠️ `orphan` | code/behavior found that maps to **no** clause (recorded separately) |

For orphans, scan the implementation's public surface for behavior that no clause explains. Classify each:
- Matches an **Out of Scope** item → flag as **violation** (scope creep against an explicit boundary).
- Otherwise → either a missing contract clause or undocumented behavior. List it; do not guess intent.

---

## Phase 4 — Write the matrix

Write `.sdd/{feature}/CONTRACT.md`:

```markdown
# Contract Traceability Matrix — {feature}

Contract: {PRD.md | BRIEF.md}
Design map: {ARCH.md | none}
Implementation: {path}
Test run: {command} — {P passed / F failed / T total}

## Clauses

| ID | Clause | Implementation | Test | Status |
|----|--------|----------------|------|--------|
| AC-1 | <verbatim> | svc.ts:42 | user.test.ts:t_creates ✅ | ✅ covered |
| AC-2 | <verbatim> | — | — | ❌ gap |
| BR-1 | <verbatim> | order.ts:80 | order.test.ts:t_total ❌ | 🔴 failing |
| BR-2 | <verbatim> | order.ts:80 | — | 🟡 partial |

## Orphans (code with no clause)

| Code | Description | Verdict |
|------|-------------|---------|
| util.ts:90 | <what it does> | undocumented / **violation (out-of-scope)** |

## Summary

- Coverage: X/N clauses ✅ (P%)
- Failing: list of 🔴 IDs
- Gaps: list of ❌ IDs
- Partial: list of 🟡 IDs
- Orphans: count
```

---

## Done — Report

```
Contract verification complete for "<feature>".
Test suite: P passed / F failed / T total.
N clauses: ✅ X covered · 🔴 V failing · 🟡 Y partial · ❌ Z gaps · ⚠️ W orphans
Coverage: P%

Failing: BR-1         ← implementation violates contract, fix first
Gaps:    AC-2, AC-4   ← implement these
Partial: BR-2         ← add tests
Orphans: 2            ← reconcile with contract or remove

Note: a passing test confirms the clause is exercised, not that the test asserts
the right thing. Run /tdd for deeper behavioral confidence.
Matrix: .sdd/{feature}/CONTRACT.md
```

---

## Rules

- **Non-mutating.** Never edit source code or tests. The only command this skill runs is the test suite; the only file it writes is `.sdd/{feature}/CONTRACT.md`.
- Run the **full** suite, not a subset — a clause's status is only trustworthy if its test actually executed this run.
- A clause is `✅ covered` only when its test **passed** in this run; never mark covered from test existence alone.
- Every clause keeps its verbatim text and a stable ID — the matrix must be re-runnable and diff-able.
- Cite the most specific `file:line`; never claim coverage without a concrete site.
- Never invent intent for orphans — record what the code does and flag it for the user to reconcile.
- Always state the residual limitation (passing ≠ asserting the right thing) in the report.
