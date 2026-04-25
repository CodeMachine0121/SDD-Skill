---
name: prd
description: >
  Conduct a requirements interview with the user, then produce two outputs:
  (1) update the project's UL-MAP.md with newly discovered domain terms,
  (2) generate a PRD file under .sdd/ in the project root.
  Triggered by: "prd", "write a PRD", "feature spec", "product requirement", "/prd".
---

## Pre-flight

1. Locate `UL-MAP.md` in the project (check root, `docs/`, `.sdd/`).
2. **If not found** → stop. Tell the user:
   > "No UL-MAP.md found. Please run the `ubi` skill to initialize a Ubiquitous Language Map first."
3. **If found** → read it. Extract:
   - Domain terms (Section 1 — Nouns & Concepts)
   - Business actions (Section 2 — Actions & Processes)
   - Project name and bounded context (header)

   Store these internally as **domain vocabulary** to anchor interview questions in the correct business language.

---

## Interview

Ask questions **one section at a time**. Use the domain vocabulary extracted from UL-MAP.md to phrase questions in the project's established language (e.g., use the confirmed Domain Term, not the raw technical name).

Work through the table below in order. Stop after each section to wait for the user's answer before proceeding.

| # | Section | Key Questions |
|---|---------|---------------|
| 1 | **Background & Goal** | What problem does this feature solve? What outcome (metric/KPI) signals success? What is explicitly out of scope? |
| 2 | **User Personas** | Who are the primary users (use roles from UL-MAP if available)? In what context (device, timing, environment) do they act? |
| 3 | **User Stories** | Walk through each user type: what do they need to do, and why? Aim for 1–5 core stories. |
| 4 | **Business Flow & Edge Cases** | What is the happy path? What are the failure modes — interrupted flow, bad data, network errors? Any state transitions or permission rules? |
| 5 | **UI/UX** | Are there existing designs or prototypes? Any loading / empty / error states to specify? |
| 6 | **Non-Functional Requirements** | Performance targets? Security constraints? Device/browser compatibility? Analytics events? |
| 7 | **Dependencies & Risks** | Third-party services? Dependencies on other teams? Unresolved technical or legal risks? |

**Clarification rules:**
- If an answer introduces a term not in UL-MAP.md, note it for the update step.
- If an answer is ambiguous, ask one targeted follow-up before moving on.
- If the user says "skip" or "N/A" for a section, record it as `N/A` and continue.

---

## Output — Step 1: Update UL-MAP.md

After the interview is complete:

1. Identify new terms, actions, or enum values surfaced during the interview.
2. Apply `UPDATE` mode from the `ubi` skill:
   - Add new rows; never delete existing rows.
   - Mark new entries with status `Archeology` unless the user confirmed the business meaning during the interview (then use `Confirmed`).
   - Update `Last Updated` date.
3. Report: "Added N term(s) to UL-MAP.md."

---

## Output — Step 2: Generate PRD

1. Determine the feature name (slug) from the user's requirement (e.g., `user-login`, `export-report`).
2. Ensure `.sdd/` directory exists at the project root (create if absent).
3. Write `.sdd/<feature-slug>-PRD.md` using [references/prd-template.md](references/prd-template.md).
   - Fill every section with answers collected during the interview.
   - Use domain vocabulary from UL-MAP.md for all entity and action names.
   - Mark any unanswered section `TBD` rather than leaving it blank.
4. Report: "PRD written to .sdd/<feature-slug>-PRD.md."
