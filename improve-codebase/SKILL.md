---
name: improve-codebase
description: >
  Improve the design quality of an existing codebase by identifying fragmented logic
  and consolidating it into deep modules — simple interfaces hiding complex internals.
  Triggered by: "improve codebase", "deep module", "/improve-codebase".
---

## Goal

Find scattered, related code and consolidate it into **deep modules**:
small, stable interfaces that absorb complexity so callers stay simple.
See [references/deep-module.md](references/deep-module.md) for principles and diagnostics.

---

## Phase 1 — Explore

Scan the codebase to understand the existing structure.

1. Ask the user for scope (single service / layer / whole project).
2. Map public interfaces and their callers.
3. Note code blocks that are logically related but currently spread across different places.
4. Flag areas where callers have to sequence multiple calls to complete one business action — these are the shallow spots.

---

## Phase 2 — Find Opportunities

From the exploration, identify candidate modules to consolidate:

- Code blocks that belong together conceptually but live apart structurally.
- Interfaces whose complexity makes the system hard for humans (or AI) to navigate.
- Any boundary where the caller knows more about *how* than it should.

Present candidates to the user. Confirm scope before proceeding.

---

## Phase 3 — Wrap in a Deep Module

For each confirmed candidate:

1. **Collect** the related code blocks into one module.
2. **Design a simple interface** — one method per business action, ≤ 2 params, verb-phrase name.
3. **Hide the implementation** — move all sequencing, validation, and decisions inside.
4. The module absorbs the complexity; callers see only the result.

---

## Phase 4 — Create Testable Boundaries

Once the module boundary is clear:

1. **Write interface tests** — test through the public interface only.
2. **Validate from outside** — treat the module as a grey box: verify inputs and outputs, not internal steps.
3. Humans review the interface design and test results; they do not need to audit internal implementation line-by-line.

Run all existing tests. Stop and report if any fail.

---

## Done

Report: "Improve complete. N modules deepened, M tests added."
