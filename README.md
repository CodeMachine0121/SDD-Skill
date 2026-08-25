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
ubiquitous-language-mapping  →  clarify  →  prd  →  architecture  →  implement  →  improve-codebase
                                                                     └ tdd │ code-first
 ↑                    ↑                      ↑
 └─── foundation ─────┴── anytime, standalone ┘
```

**The `.sdd/` folder** (Software Design Documents) is the shared workspace — skills read from and write to it, giving you a living record of domain knowledge, requirements, and decisions.

> 📊 **Visual guide:** open [`docs/workflow.html`](docs/workflow.html) for a single-file, self-contained page explaining each stage's purpose, inputs, and outputs.

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
- Maps every PRD scenario to the component that fulfills it (traceability for `/contract` and `/implement`)
- Never edits source code; outputs `.sdd/{date}-{feature}/ARCH.md`

---

### `implement` — Implementation Entry Point (pick your discipline)
The stage where the feature actually gets built. Reads the same two documents as `tdd` — `PRD.md` and `ARCH.md` — and lets you choose **how** to get there:

- **TDD mode** — hands off to the `tdd` skill: red-green-refactor, one case at a time
- **Code-first mode** — build the slice whole, then cover it (Ousterhout's "write the unit, then test it")

Why the choice exists: TDD's small steps are a **human** discipline — they shrink the problem until it fits in *our* working memory. Uncle Bob, TDD's loudest advocate, has [said so himself](https://www.youtube.com/watch?v=zcLPGC-tvgk): that constraint is about human short-term memory, and imposing it on a model that holds the entire feature at once is not obviously wise — provided you still land at 100% coverage.

What code-first mode refuses to give up is the three guarantees TDD hands you for free:

| TDD gives free | Code-first earns it back |
|---|---|
| Test derived from the requirement (no code exists to copy from) | **Oracle phase** — expected outcomes pinned from the PRD *before* any implementation, then frozen |
| RED proves the test can fail | **Falsify phase** — break the behavior, watch that test go red, restore (or run mutation testing) |
| Every line exists because a test demanded it | **Coverage gate** — 100% of new/changed lines and branches, or the code gets deleted |

Without those, "test-after" degrades into tests that describe whatever the code already does — green, worthless, and worse than nothing because they license the next change. Per-feature choice, with a recommendation (algorithmic / money / legacy → TDD; settled design, broad mechanical slice → code-first), and a bail-out back to red-green for any case whose expected outcome you can't state up front.

---

### `tdd` — Test-Driven Implementation
The red-green-refactor engine — callable directly, or as `implement`'s TDD mode. Implements a feature against exactly two documents — `PRD.md` (acceptance criteria) and `ARCH.md` (the technical design) — through red-green-refactor cycles.

1. Parses the PRD's Gherkin scenarios into atomic test cases, mapped to the components in `ARCH.md`
2. Presents the test plan for your review
3. Runs one test case at a time: write failing test → make it pass → refactor
4. **Commits every cycle** (and every app integration separately) — through the shared `commit` skill

Tracks progress in SQL so sessions can be resumed mid-cycle.

---

### `commit` — Commit Convention
The commit rules, extracted once and shared by `tdd` and `implement` so the log looks the same no matter which discipline built the feature. Also runnable standalone as `/commit` on a dirty tree.

- **Conventional Commits** — `type(scope): summary`, imperative, scope in UL-Map vocabulary
- **One commit per unit of work** — a completed TDD cycle (test + implementation together), or one app integration (composition root / DI / route / config) on its own
- **No SDD identifiers** — no test-case id, AC/scenario number, document filename, or feature-folder date; the log is read by people who never open `.sdd/`
- **English, always** — regardless of the language of the conversation, the spec, or the existing history, so `git log --grep` and changelog tooling keep working

Standalone `/commit` reads the diff, groups it into units of work (proposing a split when there is more than one), verifies green, stages by path, and commits — no push, no PR, unless asked.

---

### `contract` — Contract Traceability Matrix
Verifies that an implementation conforms to the contract — the feature's **PRD** (or **BRIEF**) — using the **Acceptance Criteria as the oracle**, not the test suite's pass/fail. For every clause it first derives the expected outcome from the spec alone, then judges **independently** whether the test asserts that outcome and whether the production code produces it. A clause conforms only when both hold.

- Reads the contract from `.sdd/{feature}/` — `PRD.md` preferred, else `BRIEF.md`
- **Oracle first:** derives each clause's expected result from the Gherkin before opening any code, so the answer can't be rationalized backward from the implementation
- Audits the test (asserts-oracle / mis-asserted / shallow) and the code (produces-oracle / diverges / unclear) separately against that oracle
- Flags **violations** (code produces the wrong outcome), **mis-asserted** clauses (green test asserts the wrong/weaker thing), **gaps**, **partial**, and **orphans** (code answering to no clause — including out-of-scope violations)
- Uses `ARCH.md`'s scenario→component map (when present) to locate code faster
- Never edits source code; outputs `.sdd/{feature}/CONTRACT.md`

> A **static conformance audit** — it judges test assertions and code paths against the spec, not by running the full suite. For dynamic proof of an `unclear` clause, drive it via `/implement`.

---

### `visionize` — Plan Visualization
Turns a feature's `BRIEF.md` and/or `PRD.md` into a single self-contained HTML page so the team can *see* the plan instead of reading it — goal, scope, personas, user stories, business flow, and risks rendered as diagrams and cards.

- Reads the spec from `.sdd/{date}-{feature}/` — `PRD.md` and/or `BRIEF.md`
- Renders a Mermaid flow diagram, scope split, priority-grouped story cards with their **Gherkin scenarios** (Given/When/Then, tagged happy/boundary/exception), and a risk list
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
7. /implement         ← pick TDD or code-first, build it → green suite
```

> Step 7 asks which discipline to use, and recommends one for that feature. `/implement tdd` and `/implement code-first` skip the question; `/tdd` still works as a direct entry point.

### Ongoing maintenance (anytime, not feature-bound)

```
/prd project          ← refresh PROJECT.md when architecture evolves
/improve-codebase     ← consolidate fragile or scattered code
/commit               ← group, verify and commit whatever is in the tree
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
| `implement` | `/implement`, `/implement code-first` | implementation + tests (mode of your choice) |
| `tdd` | `/tdd` | implementation + tests (red-green-refactor) |
| `commit` | `/commit` | commits, English Conventional Commits, one per unit of work |
| `contract` | `/contract` | `.sdd/{feature}/CONTRACT.md` |
| `improve-codebase` | `/improve-codebase` | refactored modules + interface tests |
