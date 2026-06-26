---
name: visionize
description: >
  Turn a feature's BRIEF.md and/or PRD.md into a single self-contained HTML page
  that visualizes the plan — goal, scope, personas, user stories, business flow,
  acceptance criteria, and risks — using diagrams and charts so developers can
  see what is being built at a glance. Never edits source code.
  Triggered by: "visionize", "visualize the plan", "/visionize".
---

## What this produces

One **self-contained `.html` file** (`VISION.html`) that renders the feature plan
visually: a flow diagram for the business logic, cards for personas and user
stories, an in/out scope split, a priority/roadmap view, and a risk list. It is a
*read-only projection* of the existing spec — it adds no new requirements and
makes no decisions the BRIEF/PRD did not already record.

The page must open correctly by double-clicking the file (no build step, no local
server). Diagrams use **Mermaid via its CDN**; everything else is inline HTML/CSS.
If a clause is missing from the source, render it as a clearly-marked `TBD` rather
than inventing content.

---

## Pre-flight

1. Find the spec source under `.sdd/{date}-{feature}/`:
   - Read **`PRD.md`** if present (richer: personas, user stories, ACs, flow, risks).
   - Read **`BRIEF.md`** if present (goal, requirements, out-of-scope, open decisions).
   - Read **both** when both exist — PRD is authoritative, BRIEF fills gaps PRD left out.
   - Neither found → stop: "No spec found. Run `/clarify` and `/prd` first."
   - Multiple feature folders → ask the user which one.
2. Read `.sdd/UL-MAP.md` if present — use confirmed domain terms for labels in the
   visualization so the page speaks the project's language. Missing → continue.
3. Read `.sdd/PROJECT.md` if present — use its vision/tech-stack only as context for
   framing; do not surface project-level content as feature scope.

---

## Phase 1 — Extract the plan model

Parse the source(s) into a structured model. Pull only what the spec states; mark
anything absent as `TBD`. Map each source section to a visual block:

| Source (PRD §  / BRIEF) | Plan element | Visualization |
|---|---|---|
| Background & Goal / Goal | Problem + expected outcome | Hero banner + outcome/KPI callouts |
| Out of Scope | Scope boundary | Two-column **In scope / Out of scope** split |
| User Personas | Who & context | Persona cards |
| User Stories & Acceptance Criteria | What & done-criteria | Story cards grouped by **priority (P0/P1/P2)**, each with an AC checklist |
| Business Flow & Logic | Happy path + edge cases | **Mermaid `flowchart`** (happy path solid, edge cases dashed) |
| Non-Functional Requirements | Constraints | Badge/metric grid (perf, security, compatibility) |
| Dependencies & Risks / Open Decisions | Risks & unknowns | Risk list with severity, open-decision callouts |

Keep clause text close to verbatim so the page is traceable back to the spec.

If a business flow is not explicitly described, derive a minimal flow **only** from
the stated user stories' happy paths, and label the diagram "Derived from user
stories" so the developer knows it was inferred.

---

## Phase 2 — Render the HTML

Build `VISION.html` from [references/html-template.html](references/html-template.html).
Fill each block from the Phase 1 model. Requirements for the output file:

- **Self-contained:** all CSS and JS inline. The only external resource is the
  Mermaid script from its CDN (`https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js`),
  initialized with `mermaid.initialize({ startOnLoad: true })`.
- **Responsive & printable:** readable on a laptop and when printed to PDF; wide
  diagrams scroll inside their own container, never the page body.
- **Title** = the feature name. Show the source(s) used (`PRD.md` / `BRIEF.md`) and
  the feature date in a small header line.
- **No invented content:** every rendered statement traces to the spec; gaps show
  as a muted `TBD` chip.
- Use UL-MAP domain terms for entity/action labels wherever they apply.

---

## Output

1. Write `.sdd/{date}-{feature}/VISION.html`.
2. Report:

```
Vision page written for "<feature>".
Source: PRD.md + BRIEF.md   (or whichever existed)
Blocks rendered: goal · scope · N personas · M stories · flow · K risks
TBD blocks: <list any that were empty in the spec>

Open it: open .sdd/{date}-{feature}/VISION.html
```

---

## Rules

- **Non-mutating.** Never edit source code, the BRIEF, or the PRD. The only file this
  skill writes is `.sdd/{date}-{feature}/VISION.html`.
- **Projection, not authoring.** Visualize only what the spec already states; never
  add requirements, infer scope, or resolve open decisions. Inferred flow must be
  labelled as derived.
- **One self-contained file.** No build step, no bundler, no local server — it must
  open by double-click. Only the Mermaid CDN script may be external.
- **Traceable.** Keep clause wording close to verbatim; mark missing content `TBD`
  rather than filling it in.
- **Speak the domain.** Use UL-MAP terms for labels when available.
