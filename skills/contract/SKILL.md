---
name: contract
description: >
  Verify an implementation conforms to its contract — the feature's PRD.md
  (preferred; its Gherkin acceptance-criteria scenarios) or BRIEF.md — using the
  Acceptance Criteria as the oracle. For each clause, first derive the expected
  outcome from the spec alone, then judge independently whether the test asserts
  that outcome and whether the production code produces it. Emits a Traceability
  Matrix with a per-clause conformance verdict; flags gaps, mis-asserted/shallow
  tests, behavior violations, and orphans. Never edits source code.
  Triggered by: "contract", "trace contract", "verify contract", "/contract".
---

## What this verifies

Behavioral conformance judged against the **Acceptance Criteria as the oracle** —
*not* against the test suite's pass/fail. For every clause the skill first derives
the concrete expected outcome from the spec alone, then checks two things
**independently** against that oracle:

- **Does the test assert the oracle?** A green test that asserts the *wrong* thing,
  or a weaker/vacuous thing, does not count.
- **Does the production code produce the oracle?** Traced by reading the code path
  the scenario exercises.

A clause conforms only when **both** hold. This is exactly what "run the suite and
see green" cannot tell you: a passing test can assert the wrong thing, and this
matrix surfaces that.

### Ceiling (state this in the report)

This is a **static conformance audit**. It reasons about the test code and the
production code against the oracle; it does **not** write new probes or execute
scenarios it invents — that would mutate the project and belongs to `/tdd`. It
*may optionally* run the single test already mapped to a clause as a corroborating
signal, but the verdict comes from the oracle comparison, never from pass/fail.

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

Present the extracted clause list to the user for confirmation before continuing (add / remove / reword).

---

## Phase 2 — Derive the spec oracle (before reading any code)

For each clause, write down its **oracle** — the concrete, checkable expected
outcome — using **only the spec text**. This step is the independence gate.

The oracle stays at the spec's own altitude: a **business-observable outcome**, not
a technical representation. The PRD is a business spec — it names no error codes,
return shapes, or field names, and neither does the oracle. "Concrete" means
*unambiguous and checkable*, not *technical*.

- For a Gherkin `Scenario`: `Given` is the precondition/state, `When` is the
  trigger, `Then` is the exact observable outcome. Restate `Then` as a specific,
  checkable business result — a state, a message, a value the domain cares about.
  *Example:* `Then 結帳被拒絕,並顯示「金額不正確」` → oracle = `結帳被拒絕 且 出現「金額不正確」的提示`.
  Do **not** invent the error code / return shape here; that bridge happens in Phase 3.
- For a business rule / example / requirement: state the expected result in the
  same concrete, business-observable form.

**Hard rule for this phase:** do **not** open the test code or production code
yet. The oracle must come from the spec, so it cannot be rationalized backward
from whatever the implementation happens to do. Record each oracle in the matrix
so it is auditable.

---

## Phase 3 — Trace and judge each clause against its oracle

Now — and only now — locate the code and test, then make **two independent
judgments**, each measured against the Phase 2 oracle (never test-vs-code):

0. **Bridge the oracle to its concrete form.** The Phase 2 oracle is a business
   outcome (e.g. `出現「金額不正確」的提示`). Use **UL-MAP** (business term ↔ code
   name) and **ARCH.md** (the return/error shapes it designed) to translate it into
   the concrete artifact you can check in code — e.g. `「金額不正確」` maps to error
   code `INVALID_AMOUNT`. This translation lives here, not in the PRD or the oracle.
   If neither UL-MAP nor ARCH pins the concrete form, note it and judge against the
   business outcome as best you can.
1. **Locate implementation** — which code satisfies the clause? Record `file:line`
   (most specific site). Start from `ARCH.md`'s mapped component if present;
   otherwise bridge business wording → code names via UL-MAP.
2. **Locate the test** — which test claims this clause? Record its name / `file:line`.
3. **Test audit — does the test assert the oracle?**
   | Value | Meaning |
   |---|---|
   | `asserts-oracle` | the test asserts exactly the oracle |
   | `mis-asserted` | the test asserts a **different** outcome than the oracle |
   | `shallow` | the test exists but does **not** pin the oracle — it would still pass if the `Then` outcome were wrong (e.g., only checks "no error thrown", or asserts a weaker condition) |
   | `no-test` | no test claims this clause |
4. **Code audit — does the production code produce the oracle?** Read the code path
   the clause's `Given/When` exercises and trace its result.
   | Value | Meaning |
   |---|---|
   | `produces-oracle` | the code path yields the oracle |
   | `diverges` | the code path yields a **different** outcome |
   | `unclear` | cannot be determined by reading (branchy, external state) |
   | `not-implemented` | no code path satisfies the clause |
5. **Optional corroboration** — you MAY run *only* the test mapped to this clause.
   A red result on an `asserts-oracle` test confirms a `diverges`. Never run the
   full suite as the basis of a verdict.

Then derive the clause's **Status**:

| Status | Meaning |
|---|---|
| ✅ `conforms` | `asserts-oracle` **and** `produces-oracle` |
| 🔴 `violation` | code `diverges` — the behavior is actually wrong; fix first (its test is necessarily wrong too) |
| 🟠 `mis-asserted` | code `produces-oracle` but the test is `mis-asserted` or `shallow` — the green light is not trustworthy; fix the test |
| 🟡 `partial` | code `produces-oracle` but `no-test` |
| ❌ `gap` | `not-implemented` |
| ❔ `unclear` | code audit is `unclear` — cannot judge by reading; confirm via `/tdd` |
| ⚠️ `orphan` | code/behavior mapping to **no** clause (recorded separately) |

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
Oracle: Acceptance Criteria ({N} clauses)

## Clauses

The `Spec-expected` column holds the business-observable oracle from Phase 2; the
concrete artifact it bridges to (via UL-MAP/ARCH) is what the audit columns check.

| ID | Clause | Spec-expected (oracle) | Impl | Test | Test audit | Code audit | Status |
|----|--------|------------------------|------|------|------------|------------|--------|
| AC-1 | <verbatim> | 結帳被拒絕,顯示「金額不正確」 | svc.ts:42 | order.test.ts:t_neg | asserts-oracle | produces-oracle | ✅ conforms |
| AC-2 | <verbatim> | ... | — | — | no-test | not-implemented | ❌ gap |
| AC-3 | <verbatim> | 顯示「金額不正確」 | svc.ts:50 | order.test.ts:t_neg2 | shallow | produces-oracle | 🟠 mis-asserted |
| BR-1 | <verbatim> | 訂單總額 = 小計 − 折扣 | order.ts:80 | order.test.ts:t_total | asserts-oracle | diverges | 🔴 violation |

## Orphans (code with no clause)

| Code | Description | Verdict |
|------|-------------|---------|
| util.ts:90 | <what it does> | undocumented / **violation (out-of-scope)** |

## Summary

- Conforms: X/N clauses ✅ (P%)
- Violations: list of 🔴 IDs  (code produces the wrong outcome)
- Mis-asserted: list of 🟠 IDs  (green test asserts the wrong/weaker thing)
- Partial: list of 🟡 IDs  (no test asserts the oracle)
- Gaps: list of ❌ IDs
- Unclear: list of ❔ IDs
- Orphans: count
```

---

## Done — Report

```
Contract verification complete for "<feature>".
Oracle: PRD Acceptance Criteria — N clauses.

✅ X conforms · 🔴 V violations · 🟠 M mis-asserted · 🟡 Y partial · ❌ Z gaps · ❔ U unclear · ⚠️ W orphans
Conformance: P%

Violations:   BR-1        ← code produces the wrong outcome, fix first
Mis-asserted: AC-3        ← green test asserts the wrong/weaker thing, fix the test
Gaps:         AC-2, AC-4  ← implement these
Partial:      BR-2        ← add a test that asserts the oracle
Unclear:      AC-7        ← can't judge by reading; confirm via /tdd
Orphans:      2           ← reconcile with contract or remove

Note: static conformance audit against the Acceptance Criteria — it judges test
assertions and code paths against the spec's expected outcome, not by running the
full suite. For dynamic proof of an unclear clause, drive it via /tdd.
Matrix: .sdd/{feature}/CONTRACT.md
```

---

## Rules

- **Non-mutating.** Never edit source code or tests. You may run *only* the single test already mapped to a clause as corroboration; the verdict comes from the oracle comparison, not pass/fail. Never depend on running the full suite. The only file this skill writes is `.sdd/{feature}/CONTRACT.md`.
- **Oracle first.** Derive each clause's expected outcome from the spec (Phase 2) **before** opening any test or production code. Never rationalize the oracle backward from the implementation.
- **Keep the oracle at the spec's altitude.** The oracle is a business-observable outcome (state / message / value the domain cares about) — never an error code, return shape, or field name. Bridge it to the concrete artifact in Phase 3 via UL-MAP + ARCH, never earlier.
- **Judge independently.** Audit the test against the oracle and the code against the oracle — never by comparing test to code (both can be wrong in the same way and pass each other).
- A clause is `✅ conforms` **only** when the test `asserts-oracle` **and** the code `produces-oracle`. A green test alone never earns `conforms`.
- Every clause keeps its verbatim text, a stable ID, and its recorded oracle — the matrix must be re-runnable and diff-able.
- Cite the most specific `file:line`; never claim implementation or a test without a concrete site.
- Never invent intent for orphans — record what the code does and flag it for the user to reconcile.
- Always state the ceiling (static audit; does not execute invented scenarios) in the report.
