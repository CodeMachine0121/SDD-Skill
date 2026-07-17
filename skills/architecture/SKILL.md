---
name: architecture
description: >
  Phase between PRD and TDD. Reads a feature's PRD.md and designs the technical
  approach — the change scope and the new classes/modules with their
  responsibilities — from the perspective of the next engineer who has to extend
  it, so future requirements land with minimal load. Outputs ARCH.md.
  Never edits source code.
  Triggered by: "architecture", "arch", "design architecture", "/architecture", "/arch".
---

## Core Rule

**Never edit source files.** This is a design stage, not an implementation stage. The only output is a design document written to `.sdd/`. The only accepted confirmation phrases are **"Confirm"** or **"Go"**.

**This is where "how" lives.** `clarify` and `prd` are deliberately business-only (*what*). This skill is the opposite: it is the technical design, so it **may and should** name concrete modules, services, classes, methods, and patterns. Its job is to translate the PRD's business acceptance criteria into a build plan.

**Design for the next engineer.** Assume someone else picks this up next sprint and has to add the *next* requirement. Optimize the design so that addition costs as little as possible: prefer **deep modules** (simple interface, complex internals hidden), isolate the axis of change behind a stable abstraction, and avoid decisions that callers would later have to unwind. See [../improve-codebase/references/deep-module.md](../improve-codebase/references/deep-module.md).

---

## Design Loop

### Step 1 — Pre-flight (read the contract & the ground)

Do **not** open any editor tool — this step is read-only.

1. **Locate the PRD.** Read `.sdd/{yyyy-MM-dd}-{feature-slug}/PRD.md`.
   **If not found** → stop and tell the user: *"No PRD found. Run `/prd` first — architecture designs against the PRD's acceptance criteria."*
2. **Read the PRD's acceptance criteria** (the Gherkin scenarios) as the design contract. Every scenario must be answerable by some component in your design; note them as the traceability targets.
3. **Read project context** if present: `.sdd/PROJECT.md` (tech stack, architecture principles, conventions) and `.sdd/UL-MAP.md` (domain vocabulary — reuse these names for classes and modules).
4. **Scan the codebase.** Map the existing structure the feature will touch: current layers/modules, entities, services, and the patterns already in use. The design must fit the grain of what exists, not invent a parallel style.
5. **Read sibling `ARCH.md`** files in other `.sdd/` feature folders for patterns and decisions already established — reuse them instead of re-deciding.

---

### Step 2 — Analyze & Design

Work out, in this order:

1. **Change scope.** Which existing components are touched, which are added, and — just as important — which are explicitly *not* touched. Keep the blast radius small and justified.
2. **New components.** For each new class/module, state its single responsibility (its *purpose*), its collaborators, and the PRD scenario(s) it exists to satisfy. Every new class must earn its place; if two proposed classes share one reason to change, merge them.
3. **Depth check.** For every new or modified boundary, run the diagnostic questions in [../improve-codebase/references/deep-module.md](../improve-codebase/references/deep-module.md). Reject shallow interfaces (caller sequences steps, names with "And/Then", growing parameter lists).
4. **The axis of change.** Ask: *what is the most likely next requirement, and where would it hit?* Put a seam there — a strategy point, a stable interface, a config surface — so that requirement becomes an addition, not a rewrite. This is the heart of "minimal load for the next engineer" and gets its own section in the output.
5. **Traceability.** Map each PRD acceptance-criteria scenario to the component(s) that fulfill it, so `/contract` and `/tdd` can verify coverage.

---

### Step 3 — Present for Confirmation

Present a concise draft using the format in [references/arch-summary-format.md](references/arch-summary-format.md): the design goal, the change-scope summary, the new-components table, and the key extensibility decisions.

If the user's feedback changes scope or approach → revise and re-present. Wait for **"Confirm"** or **"Go"**.

---

### Step 4 — Output

Only after receiving **"Confirm"** or **"Go"**:

1. Write `.sdd/{yyyy-MM-dd}-{feature-slug}/ARCH.md` using [references/arch-template.md](references/arch-template.md). Use the same feature folder as the PRD.
2. Fill every section; use domain vocabulary from UL-MAP.md for names. Mark anything undecided `TBD` rather than leaving it blank.
3. Report: "Architecture written to `.sdd/{yyyy-MM-dd}-{feature-slug}/ARCH.md`. Run `/tdd` to implement against it."

---

## Hard Constraints

| Constraint | Rule |
|---|---|
| No source edits | `Edit`, `Write`, `Bash` on source files are permanently blocked — the only file written is `ARCH.md` |
| PRD required | Refuse to design without a `PRD.md`; the acceptance criteria are the contract |
| Cover every scenario | Every PRD acceptance-criteria scenario maps to at least one component in the design |
| Justify the scope | Every touched or added component states why it exists; list what is deliberately *not* changed |
| Deep modules | New boundaries pass the deep-module diagnostics; reject shallow interfaces |
| Design for extension | The design names the likely axis of change and the seam that absorbs it |
| Confirm before writing | `ARCH.md` is written only after "Confirm" or "Go" |
