# sdd-skills

> A Claude Code plugin whose skills guide software development from domain language all the way to production-ready code.

---

## Inspiration

This project was sparked by Matt Pocock's talk **["It Ain't Broke: Why Software Fundamentals Matter More Than Ever"](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=24s)**.

The core argument: AI coding tools are powerful, but the developers who succeed with them are the ones who double down on engineering fundamentals — not the ones who try to skip them. Ubiquitous Language, vertical slices, TDD, and deep modules don't become less important when AI is in the loop. They become *more* important, because AI can produce spaghetti faster than any human ever could.

> "The devs who succeed aren't the ones who delegate everything or nothing to AI. They're the ones who fall back on engineering fundamentals... the principles that make it work didn't break. They got more important."
> — Matt Pocock

Forge turns those fundamentals into a concrete, skill-by-skill workflow for the [SDD ecosystem](https://coding-afternoon.com/gsi-protocol/).

---

## Overview

Forge provides a structured, repeatable development workflow powered by Claude Code skills. Each skill owns one phase of the process, keeping responsibilities sharp and handoffs clean.

```
ubiquitous-language-mapping  →  clarify  →  prd  →  architecture  →  tdd  →  improve-codebase
 ↑                    ↑                      ↑
 └─── foundation ─────┴── anytime, standalone ┘
```

**The `.sdd/` folder** (Software Design Documents) is the shared workspace — skills read from and write to it, giving you a living record of domain knowledge, requirements, and decisions.

---

## Skills

### `ubiquitous-language-mapping` — Ubiquitous Language Map
**Run this first.** Scans the codebase and creates `.sdd/UL-MAP.md` — a domain glossary that bridges business terms, code names, and UI labels.

- `INIT`: generate a fresh map from existing source files
- `UPDATE`: add, refine, or deprecate terms as the domain evolves

All downstream skills depend on this map to phrase questions and generate artifacts in the project's own language.

---

### `clarify` — Requirements Brief
Reaches full intent consensus with you before a single line is written. Once the intent is captured, it pins the behavior down with **concrete examples** — covering the happy path plus every boundary and exception — and folds them into a structured `BRIEF.md` written **purely in business language**, ready to hand to PRD.

- Runs a structured consensus loop (Investigate → Questions → Examples → Proposal)
- **Specification by Example:** every behavior rule is agreed via real examples; each example carries only the data that affects the behavior (data minimality)
- **Business language only:** no technical terms in the brief — domain vocabulary only, so the spec stays about *what* not *how*
- Never edits source files
- Outputs `.sdd/{date}-{feature}/BRIEF.md`

---

### `prd` — Product Requirements Document
Conducts a requirements interview grounded in the UL Map, then generates a formal PRD. Two modes:

- **Feature mode** (`/prd`): interview → PRD for a specific feature; also updates UL-MAP with any new terms. The PRD stays a **business specification** (no service/class/method names) and turns the brief's requirement examples into **Gherkin acceptance criteria** inline — no separate `.feature` file
- **Project mode** (`/prd project`): analyze architecture + UL-MAP → produce/update `.sdd/PROJECT.md` (vision, tech stack, conventions)

> **Standalone use:** run `/prd project` anytime to keep the project overview in sync with reality — no feature needed.

---

### `architecture` — Technical Design
The **"how"** stage that PRD deliberately leaves out. Reads the feature's `PRD.md` and designs the build plan: the change scope, the new classes/modules and their responsibilities (as a table), and the seams that keep future requirements cheap to add.

- Reads `PRD.md` (its Gherkin acceptance criteria are the design contract), plus `PROJECT.md` / `UL-MAP.md` and the existing codebase
- Designs from the perspective of **the next engineer** — deep modules, stable interfaces, an explicit axis-of-change seam so the next requirement is add-only
- Maps every PRD scenario to the component that fulfills it (traceability for `/contract` and `/tdd`)
- Never edits source code; outputs `.sdd/{date}-{feature}/ARCH.md`

---

### `tdd` — Test-Driven Implementation
Reads a PRD and drives implementation through red-green-refactor cycles.

1. Parses acceptance criteria into atomic test cases
2. Presents the test plan for your review
3. Runs one test case at a time: write failing test → make it pass → refactor

Tracks progress in SQL so sessions can be resumed mid-cycle.

---

### `contract` — Contract Traceability Matrix
Verifies that an agent's implementation conforms to the contract — the feature's **PRD** (or **BRIEF**) — by **executed coverage**. Builds a matrix mapping every contract clause (PRD acceptance criteria & business rules, or BRIEF requirements) to its code site and test, runs the full suite once, then flags **gaps** (clauses with no implementation), **failing** clauses (test exists but red), and **orphans** (code answering to no clause — including out-of-scope violations).

- Reads the contract from `.sdd/{feature}/` — `PRD.md` preferred, else `BRIEF.md`
- Runs the test suite but never edits source code
- Outputs `.sdd/{feature}/CONTRACT.md`

> A passing test confirms a clause is exercised, not that it asserts the right thing — pair with `/tdd` for deeper behavioral confidence.

---

### `visionize` — Plan Visualization
Turns a feature's `BRIEF.md` and/or `PRD.md` into a single self-contained HTML page so the team can *see* the plan instead of reading it — goal, scope, personas, user stories, business flow, and risks rendered as diagrams and cards.

- Reads the spec from `.sdd/{date}-{feature}/` — `PRD.md` and/or `BRIEF.md`
- Renders a Mermaid flow diagram, scope split, priority-grouped story cards, and a risk list
- Never edits source code; produces a single double-click-to-open file
- Outputs `.sdd/{date}-{feature}/VISION.html`

> A projection, not authoring — it visualizes only what the spec already states and marks anything missing as `TBD`.

---

### `improve-codebase` — Deep Module Refactoring
Improves design quality by finding scattered logic and consolidating it into deep modules — stable interfaces that absorb complexity so callers stay simple.

1. Explore: map interfaces and flag shallow spots
2. Identify consolidation candidates
3. Wrap into deep modules with clean, verb-phrase interfaces
4. Add interface tests validating inputs and outputs

> **Standalone use:** run anytime to keep code quality high, independent of any active feature.

---

## Recommended Workflow

### Starting a new project

```
1. /ubiquitous-language-mapping   ← initialize the Ubiquitous Language Map
2. /prd project       ← document vision, tech stack, and conventions
```

### Building a new feature

```
3. /clarify           ← reach consensus on what to build → BRIEF.md
4. /prd               ← conduct requirements interview   → PRD.md
5. /visionize         ← visualize the plan as HTML       → VISION.html  (optional)
6. /architecture      ← design the technical approach    → ARCH.md
7. /tdd               ← implement test-by-test           → green suite
```

### Ongoing maintenance (anytime, not feature-bound)

```
/prd project          ← refresh PROJECT.md when architecture evolves
/improve-codebase     ← consolidate fragile or scattered code
/ubiquitous-language-mapping update  ← extend the domain glossary as language shifts
```

---

## Folder Structure

```
.sdd/
├── UL-MAP.md                          # Ubiquitous Language Map (ubiquitous-language-mapping)
├── PROJECT.md                         # Project overview (prd project)
└── {yyyy-MM-dd}-{feature}/
    ├── BRIEF.md                       # Requirements brief, business language + examples (clarify)
    ├── PRD.md                         # Product requirements, business spec + Gherkin AC (prd)
    ├── ARCH.md                        # Technical design (architecture)
    └── VISION.html                    # Visual plan (visionize)
```

---

## Skills Index

| Skill | Trigger | Output |
|---|---|---|
| `ubiquitous-language-mapping` | `/ul-init`, `/ul-update` | `.sdd/UL-MAP.md` |
| `clarify` | `/clarify` | `.sdd/{date}-{feature}/BRIEF.md` |
| `prd` | `/prd`, `/prd project` | `.sdd/{date}-{feature}/PRD.md` or `.sdd/PROJECT.md` |
| `visionize` | `/visionize` | `.sdd/{date}-{feature}/VISION.html` |
| `architecture` | `/architecture`, `/arch` | `.sdd/{date}-{feature}/ARCH.md` |
| `tdd` | `/tdd` | implementation + tests |
| `contract` | `/contract` | `.sdd/{feature}/CONTRACT.md` |
| `improve-codebase` | `/improve-codebase` | refactored modules + interface tests |
