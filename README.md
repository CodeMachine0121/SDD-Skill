# Forge — AI Copilot Dev Skills

> A collection of GitHub Copilot CLI skills that guide software development from domain language all the way to production-ready code.

---

## Overview

Forge provides a structured, repeatable development workflow powered by Copilot CLI skills. Each skill owns one phase of the process, keeping responsibilities sharp and handoffs clean.

```
ubiquitous-language-mapping  →  clarify  →  prd  →  tdd  →  improve-codebase
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
Reaches full intent consensus with you before a single line is written. Produces a structured `BRIEF.md` that feeds directly into PRD.

- Runs a structured clarification loop (Gathering → Questions → Proposal)
- Never edits source files
- Outputs `.sdd/{date}-{feature}/BRIEF.md`

---

### `prd` — Product Requirements Document
Conducts a requirements interview grounded in the UL Map, then generates a formal PRD. Two modes:

- **Feature mode** (`/prd`): interview → PRD for a specific feature; also updates UL-MAP with any new terms
- **Project mode** (`/prd project`): analyze architecture + UL-MAP → produce/update `.sdd/PROJECT.md` (vision, tech stack, conventions)

> **Standalone use:** run `/prd project` anytime to keep the project overview in sync with reality — no feature needed.

---

### `tdd` — Test-Driven Implementation
Reads a PRD and drives implementation through red-green-refactor cycles.

1. Parses acceptance criteria into atomic test cases
2. Presents the test plan for your review
3. Runs one test case at a time: write failing test → make it pass → refactor

Tracks progress in SQL so sessions can be resumed mid-cycle.

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
5. /tdd               ← implement test-by-test           → green suite
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
    ├── BRIEF.md                       # Requirements brief (clarify)
    └── PRD.md                         # Product requirements (prd)
```

---

## Skills Index

| Skill | Trigger | Output |
|---|---|---|
| `ubiquitous-language-mapping` | `/ul-init`, `/ul-update` | `.sdd/UL-MAP.md` |
| `clarify` | `/clarify` | `.sdd/{date}-{feature}/BRIEF.md` |
| `prd` | `/prd`, `/prd project` | `.sdd/{date}-{feature}/PRD.md` or `.sdd/PROJECT.md` |
| `tdd` | `/tdd` | implementation + tests |
| `improve-codebase` | `/improve-codebase` | refactored modules + interface tests |
