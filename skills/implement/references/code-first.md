# Code-First Playbook

Build the slice whole, then earn back — explicitly — every guarantee TDD hands you for
free. Work one **ARCH component or vertical slice** at a time; a slice is as much as
you can state the full expected behavior for before writing it.

---

## Phase 1 · ORACLE 🎯 — pin expected outcomes before any code

The single most important step. Test-after fails when the tests are written by reading
the implementation; the cure is to write down what "correct" means while the
implementation does not yet exist.

1. Parse the PRD Section 3 Gherkin scenarios into atomic cases — one behavior each,
   preserving happy / boundary / exception coverage. Add derived cases from Sections 4 & 6
   (business rules, edge cases). Skip empty/TBD rows.
2. For each case record the **expected outcome in concrete terms** — the actual return
   value, the exception type, the call that must reach the boundary. Not "returns the
   right total": `returns 315`. If the PRD does not determine it, ask the user; do not
   invent it and do not defer it to "whatever the code does".
3. Map each case to the component in `ARCH.md` that owns it, so the test lands at the
   right boundary.
4. Track in SQL: `test_cases(id TEXT PK, prd_file TEXT, user_story TEXT, description TEXT,
   expected_outcome TEXT, status TEXT DEFAULT 'pending')`. The `id` is internal tracking
   only and **never** appears in a commit message.
5. Present the list to the user for confirmation (add / remove / reword) before building.

Check for existing rows matching this PRD first → if found, ask to resume or reset.

> The oracle table is frozen once BUILD starts. Editing an expected outcome afterwards to
> match what the code produced is the exact failure this phase exists to prevent. If the
> spec turns out to be wrong, say so, fix the PRD with the user, and re-confirm the row.

---

## Phase 2 · BUILD 🔨 — implement the slice

- Implement to `ARCH.md`: its components, responsibilities, and seams. Do not invent a
  different structure because it is convenient while typing.
- Use `UL-MAP.md` vocabulary for every name.
- No tests yet. Keep the build compiling at all times.
- Write only what the pinned cases require. Speculative generality has nothing to cover
  it in Phase 5 and will have to be deleted.
- Hit a case whose expected outcome you cannot state? Stop and run that one case
  red-green (see the bail-out rule in SKILL.md), then continue.

---

## Phase 3 · COVER 🧪 — write the tests

Per [test-design.md](../../tdd/references/test-design.md): behavior through the public
API only, AAA structure, one observable outcome per test, mock only boundaries.

**The rule that makes test-after honest:**

> Every expected value in an assertion is copied from the Phase-1 oracle row — not from
> the implementation, not from a debugger, not from the test output.

If you catch yourself running the code to find out what to assert, that test is
worthless: it will pass by construction and will keep passing when the behavior breaks.
Stop, re-derive from the PRD, or ask the user.

Red flags to reject in your own tests:

| Smell | Why it is fatal here |
|---|---|
| Expected value pasted from actual output | Asserts the bug, not the requirement |
| Asserting on internals / private state | Locks in the implementation you just wrote |
| Assertion weaker than the oracle (`NotNull` where the oracle says `315`) | Green, vacuous, unfalsifiable |
| Test restates the code's control flow step by step | Mirror test — breaks on every refactor, catches nothing |

Run the full suite. All green before Phase 4.

---

## Phase 4 · FALSIFY 🔴 — prove each test can fail

TDD's RED step is not ceremony: it is the only proof that a test is connected to the
behavior it claims to cover. Reconstruct it.

**If the project has a mutation-testing tool, use it** — it does this exhaustively:

| Stack | Tool |
|---|---|
| .NET / C# | Stryker.NET (`dotnet stryker`) |
| TypeScript / JS | StrykerJS |
| Java / Kotlin | PIT (`pitest`) |
| Python | `mutmut`, `cosmic-ray` |
| Go, others | usually none → do the manual loop |

Scope it to the changed files. Target: **no surviving mutants** in the new code; any
survivor is either a missing assertion or dead code. Equivalent mutants get named and
justified in the report, not silently ignored.

**Otherwise, do it by hand, per case:**

1. Break exactly the behavior the test covers — invert a condition, return a wrong
   constant, delete the guard clause.
2. Run **that test only**. It must fail, and for the right reason.
3. Restore the code. Re-run: green.

A test that stays green while its behavior is broken is vacuous → rewrite it against the
oracle row and falsify again.

> **Never commit, never leave the session, while a deliberate break is in place.** Restore
> immediately after each check, and confirm the full suite is green before Phase 5.

---

## Phase 5 · GATE 📊 — 100% coverage of the changed code

This is the condition that makes the whole trade legitimate — Ousterhout's "test after"
is only sound if it ends at full coverage.

| Stack | Command |
|---|---|
| .NET / C# | `dotnet test --collect:"XPlat Code Coverage"` (coverlet) |
| TypeScript / JS | `vitest run --coverage` / `jest --coverage` |
| Go | `go test ./... -coverprofile=cover.out && go tool cover -func=cover.out` |
| Python | `pytest --cov=<pkg> --cov-report=term-missing` |
| Java / Kotlin | JaCoCo report |

Rules:

- **Scope to the diff.** Project-wide percentage is noise; what matters is new and
  changed lines, and their branches.
- **100% of that scope**, lines *and* branches. Every uncovered line is one of two things:
  - a behavior the oracle missed → add the case (append it to the table, write the test,
    falsify it), or
  - code nothing requires → **delete it**.
- Coverage is a floor, not a proof. Phase 4 is what makes the number mean something —
  100% coverage with vacuous assertions is 0% verification. Never present coverage as
  evidence of correctness on its own.
- **No coverage tool in the project?** Say so plainly. Fall back to a line-by-line audit
  of the diff against the case list, report it as a manual audit, and do not state a
  percentage.

---

## Phase 6 · REFACTOR 🔵

Apply [refactor.md](../../tdd/references/refactor.md). Run **all module tests** after each
move; stop if any fail. Behavior must not change — the tests from Phase 3 are now the net
that lets you move safely, which is the payoff for having written them properly.

Mark the covered cases `done` in SQL.

---

## Phase 7 · COMMIT ✅

Follow the [`commit`](../../commit/SKILL.md) skill — English Conventional Commits, one
unit of work per commit, no SDD identifiers. For this mode that means:

- One commit per slice: implementation **and** its tests together.
- **App integration on its own commit** — wiring into the composition root / DI / an
  endpoint / config, separate from behavior slices.
- Stage only this slice's files.
- Every commit green, self-contained, and free of any mutation from Phase 4.

Repeat from Phase 2 for the next slice. Mark unresolvable cases `blocked` with a reason.
