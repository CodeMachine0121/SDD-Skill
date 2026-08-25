---
name: implement
description: >
  Implement a feature against its PRD.md (acceptance criteria) and ARCH.md
  (technical design), letting the developer pick the discipline: TDD
  (red-green-refactor, one case at a time) or CODE-FIRST (build the slice whole,
  then cover it with tests to 100% of the changed code). Both modes pin the
  acceptance-criteria oracle before writing code, and both end green, covered,
  and committed.
  Triggered by: "implement", "/implement", "implement this feature",
  "code-first", "test-after".
---

## Why two modes

TDD's small steps are a **human** discipline: they exist because a person cannot hold
a whole feature — spec, design, edge cases, half-written code — in working memory at
once. Red-green-refactor shrinks the problem until it fits. Uncle Bob, TDD's loudest
advocate, has said the same out loud: that constraint is about *our* short-term
memory, and imposing it on a model whose working memory holds the entire feature at
once is not obviously wise. John Ousterhout's position — write the unit, then test
it, so long as you end at full coverage — becomes a legitimate option.

So this skill offers the choice. What it does **not** offer is a way out of the
guarantees TDD hands you for free. Writing tests after the code loses three of them,
and code-first mode buys each one back explicitly:

| TDD gives you free | Code-first must earn it back |
|---|---|
| The test is derived from the requirement (no code exists yet to copy from) | **Phase 1 pins the oracle** — expected outcomes written from the PRD *before* any implementation |
| RED proves the test can actually fail | **Phase 4 falsifies** — break the behavior, watch the test go red, restore |
| Every line exists because a test demanded it | **Phase 5 gates coverage** — 100% of the changed code, or delete the code |

Skipping any of those turns "test-after" into "tests that describe whatever the code
already does" — green, worthless, and worse than no tests because they license the
next change.

---

## Pre-flight (both modes)

The design contract is exactly two documents: **`PRD.md`** (what to build + acceptance
criteria) and **`ARCH.md`** (how it is structured).

1. Locate the feature folder `.sdd/{yyyy-MM-dd}-{feature-slug}/`. None → stop ("Run `/prd` first"). Multiple candidates → ask which feature. Then read:
   - **`PRD.md`** — the Section 3 acceptance-criteria scenarios (Gherkin) are the behavior contract.
   - **`ARCH.md`** — the change scope and the new/modified classes and modules with their responsibilities. Implement to this design; do not invent a different structure. Missing → warn ("Run `/architecture` first for a design to follow") and continue only if the user insists.
2. Find `.sdd/UL-MAP.md` (fallback: root, `docs/`). Missing → warn, continue. Use its vocabulary for names.
3. **Load the [`commit`](../commit/SKILL.md) skill** — it owns the commit convention for this workflow (English Conventional Commits, one commit per unit of work, no SDD identifiers). Follow it for every commit made in this run, in either mode.
4. **Detect the test runner and the coverage tool** (see [code-first.md](references/code-first.md) for the per-stack table). Code-first mode needs coverage; if the project has none, say so now — the gate degrades to a manual diff audit and you must not claim a percentage.

---

## Mode selection

Honor an explicit argument: `/implement tdd` → Mode A. `/implement code-first`
(aliases: `bob`, `ousterhout`, `test-after`) → Mode B.

No argument → **ask the user**, with a recommendation based on the feature at hand:

| Signal in this feature | Recommend |
|---|---|
| Algorithmic / many boundary rules / you cannot state the expected output without working it out | **TDD** |
| Money, safety, security, or anything where a wrong answer is expensive | **TDD** |
| Changing legacy code that has no tests around it | **TDD** |
| Design is settled in `ARCH.md`, the slice is broad and mechanical (CRUD, plumbing across layers, wiring) | **code-first** |
| One coherent change spanning many files, where per-case red-green would thrash the same files repeatedly | **code-first** |
| The developer is exploring — the shape of the answer is still unknown | **TDD** |

State the recommendation and the reason in one line, then let the user choose. The
choice is per feature, not per project; ask again next feature.

---

## Mode A — TDD

Invoke the **`tdd`** skill and follow it. Do not restate or fork its cycle here — it
owns red-green-refactor, its test plan, and its per-cycle commits (through the same
[`commit`](../commit/SKILL.md) convention). This skill's job ends at the handoff.

---

## Mode B — Code-first

Full playbook: [code-first.md](references/code-first.md). The loop, per ARCH component
or vertical slice:

**1 · ORACLE 🎯** — before any implementation, parse the PRD Gherkin into atomic cases and write down each one's **expected outcome derived from the spec alone**. Confirm the list with the user. This is the anti-bias guard: later, tests are checked against *this*, never against what the code turned out to do.

**2 · BUILD 🔨** — implement the slice whole, to the design in `ARCH.md`. No tests yet. Keep it compiling. Any deviation from ARCH gets raised with the user, not quietly taken.

**3 · COVER 🧪** — write the tests for every pinned case. Every assertion's expected value comes from the Phase-1 oracle text. **If you can only get an expected value by running the implementation, stop** — re-derive it from the PRD or ask. Full suite green.

**4 · FALSIFY 🔴** — the RED phase, reconstructed. For each test, break the behavior it covers (invert the condition, return a wrong constant, delete the line), confirm **that** test fails, restore. A test that survives its own mutation is vacuous → rewrite it. Never commit while mutated.

**5 · GATE 📊** — run coverage scoped to the changed code. **100% of new/changed lines and branches.** Anything uncovered → add the missing case (append it to the oracle table) or delete the code as speculative. Report the real number.

**6 · REFACTOR 🔵** — apply the moves in [refactor.md](../tdd/references/refactor.md). Run all module tests after each move; stop if any go red.

**7 · COMMIT ✅** — per the [`commit`](../commit/SKILL.md) skill: one commit per slice, implementation and its tests together; app integration (composition root / DI / endpoint / config) is its own commit. Every commit green, self-contained, and free of any leftover mutation from Phase 4.

**Bail out to TDD** for any case where you find yourself unable to state the expected
outcome up front, or debugging blind: run that one case red-green instead and say so.
Mode is a default, not a cage.

---

## Test design

Both modes write tests the same way — behavior through the public API only, per
[test-design.md](../tdd/references/test-design.md). Code-first does not license
testing internals just because the internals are already there.

---

## Done

Run the full suite. Report the mode used, then:

- **TDD** — as the `tdd` skill reports: "N done, M blocked. All tests passing."
- **Code-first** — "Code-first complete. N cases covered, M mutations falsified, coverage X% of changed lines (tool: …). All tests passing." Uncovered lines, surviving mutants, and cases bailed out to TDD are listed explicitly. Never round X up, and never state it at all if no coverage tool ran.
