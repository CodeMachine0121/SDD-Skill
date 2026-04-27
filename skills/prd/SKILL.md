---
name: prd
description: >
  Two modes: (1) FEATURE — conduct a requirements interview and generate a PRD
  file under .sdd/; (2) PROJECT — analyze UL-MAP.md and project architecture to
  produce or update .sdd/PROJECT.md (vision, tech stack, conventions).
  Triggered by: "prd", "write a PRD", "feature spec", "product requirement",
  "/prd", "/prd project", "project overview", "project map".
---

## Mode Detection

Determine mode **before** any other step:

- If the user invoked `/prd project`, or said "project overview / project map / project vision" → **PROJECT mode** (go to [Project Pre-flight](#project-pre-flight)).
- Otherwise → **FEATURE mode** (go to [Feature Pre-flight](#feature-pre-flight)).

---

## PROJECT mode

### Project Pre-flight

1. Locate `UL-MAP.md` at `.sdd/UL-MAP.md`.
2. **If not found** → stop. Tell the user:
   > "No UL-MAP.md found. Please run the `ubiquitous-language-mapping` skill first."
3. **If found** → read it. Extract:
   - Project name and bounded context (header)
   - All domain terms and business actions
   - Any existing tech or architecture notes

4. Scan project architecture signals. Read (if present):
   - `package.json` / `pyproject.toml` / `*.csproj` / `go.mod` → infer tech stack and key libraries
   - Top-level folder structure (`ls` one level deep) → infer architectural style
   - `CLAUDE.md` / `README.md` → extract stated conventions, constraints, or principles

5. Check if `.sdd/PROJECT.md` already exists.
   - **If found** → read it. This is **UPDATE mode**: preserve existing content, only fill gaps or refresh stale sections.
   - **If not found** → this is **CREATE mode**: draft from scratch.

### Project Analysis & Draft

Using data gathered in pre-flight, draft a `PROJECT.md` using [references/project-template.md](references/project-template.md):

- **Section 1 (Vision):** Leave as `TBD` — cannot be inferred from code; must come from user.
- **Section 2 (Tech Stack):** Populate from dependency files and folder scan.
- **Section 3 (Architecture Principles):** Infer style and patterns from folder structure and any existing docs.
- **Section 4 (Conventions):** Extract from `CLAUDE.md`, `README.md`, or config files (linters, formatters, CI).
- **Section 5 (Constraints):** Extract from any stated SLAs, security configs, or `CLAUDE.md` notes.
- **Section 6 (Glossary):** Pull top 5–10 most project-critical terms from UL-MAP.md.

Present the draft to the user. Clearly mark every auto-inferred field and every `TBD`.

### Project Gap Interview

Ask **only** about sections that are `TBD` or flagged as uncertain. One section at a time.

| # | Section | Key Questions |
|---|---------|---------------|
| 1 | **Vision & Mission** | What core problem does this project solve? Who are the primary users? What does success look like? What is explicitly out of scope? |
| 4 | **Conventions** (gaps only) | Any naming, branching, testing, or review rules not captured in config files? |
| 5 | **Constraints** (gaps only) | Performance SLAs? Security requirements? Budget or deployment constraints? Known tech debt? |

Skip any section already fully populated from analysis.

### Project Output

1. Write (or overwrite) `.sdd/PROJECT.md` with the completed content.
2. Report: "PROJECT.md written to `.sdd/PROJECT.md`."

---

## FEATURE mode

### Feature Pre-flight

1. Locate `UL-MAP.md` at `.sdd/UL-MAP.md`.
   Also read `.sdd/PROJECT.md` if it exists — use it to anchor answers (tech stack, conventions, constraints) without re-asking the user about project-level context.
2. **If not found** → stop. Tell the user:
   > "No UL-MAP.md found. Please run the `ubiquitous-language-mapping` skill to initialize a Ubiquitous Language Map first."
3. **If found** → read it. Extract:
   - Domain terms (Section 1 — Nouns & Concepts)
   - Business actions (Section 2 — Actions & Processes)
   - Project name and bounded context (header)

   Store these internally as **domain vocabulary** to anchor interview questions in the correct business language.

4. Check if a feature folder `.sdd/{yyyy-MM-dd}-{feature-slug}/` already exists with a `BRIEF.md` inside. **If found** → read the brief and use it to pre-fill interview answers. Tell the user which sections the brief already covers, then only ask questions for gaps.

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
2. Apply `UPDATE` mode from the `ubiquitous-language-mapping` skill:
   - Add new rows; never delete existing rows.
   - Mark new entries with status `Archeology` unless the user confirmed the business meaning during the interview (then use `Confirmed`).
   - Update `Last Updated` date.
3. Report: "Added N term(s) to UL-MAP.md."

---

## Output — Step 2: Generate PRD

1. Determine today's date in `yyyy-MM-dd` format and the feature slug (e.g., `user-login`, `export-report`).
2. Ensure the feature folder `.sdd/{yyyy-MM-dd}-{feature-slug}/` exists (create `.sdd/` and the subfolder if absent).
3. Write `.sdd/{yyyy-MM-dd}-{feature-slug}/PRD.md` using [references/prd-template.md](references/prd-template.md).
   - Fill every section with answers collected during the interview.
   - Use domain vocabulary from UL-MAP.md for all entity and action names.
   - Mark any unanswered section `TBD` rather than leaving it blank.
4. Report: "PRD written to `.sdd/{yyyy-MM-dd}-{feature-slug}/PRD.md`."
