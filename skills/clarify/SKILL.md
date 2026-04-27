---
name: clarify
description: >
  Reach 100% intent consensus with the user, then compile a requirements brief
  for handoff to PRD. Never edits source code.
  Triggered by: "clarify", "/clarify".
---

## Core Rule

**Never edit source files.** The only output this skill produces is a requirements brief written to `.sdd/`. The only accepted confirmation phrases are **"Confirm"** or **"Go"**.

---

## Consensus Loop

### Step 1 — Gathering

When the user presents a request:

1. Do **not** open any editor tool.
2. Read relevant files silently if needed to understand context.
3. Write down your understanding of the goal in plain language.

---

### Step 2 — Clarification

Identify anything that is ambiguous, under-specified, or potentially risky. Present using this exact structure:

```
**[Current Understanding]**
<one-paragraph summary of what you think the user wants>

**[Open Questions]**
1. <question>
2. <question>
…

**[Estimated Scope]**
- Files likely affected: <list>
- Files definitely NOT affected: <list if useful>
```

Wait for the user's answers before continuing.

---

### Step 3 — Proposal

After ambiguities are resolved, present the brief outline:

```
**[Proposed Brief]** — awaiting confirmation

Goal: <one sentence>

Requirements:
- <requirement>
- <requirement>

Out of scope:
- <item>

Open decisions (for PRD to resolve):
- <item>

Will write to: .sdd/<feature-name>-BRIEF.md

Type "Confirm" or "Go" to generate the file.
```

If the user's response introduces new variables or changes scope → return to **Step 2**.

---

### Step 4 — Output

Only after receiving **"Confirm"** or **"Go"**:

1. Determine today's date in `yyyy-MM-dd` format and the feature slug (e.g., `user-login`).
2. Create the feature folder `.sdd/{yyyy-MM-dd}-{feature-slug}/` (create `.sdd/` first if absent).
3. Write `.sdd/{yyyy-MM-dd}-{feature-slug}/BRIEF.md` using this template:

```markdown
# <Feature Name> — Requirements Brief

## Goal
<one-paragraph summary>

## Requirements
- <requirement>

## Out of Scope
- <item>

## Open Decisions
Items the PRD author should resolve:
- <item>

## Context / Background
<any relevant notes from the clarification conversation>
```

4. Report: "Brief written to `.sdd/{yyyy-MM-dd}-{feature-slug}/BRIEF.md`. Run `/prd` to generate the PRD."

---

## Hard Constraints

| Constraint | Rule |
|---|---|
| No source edits | `Edit`, `Write`, `Bash` on source files are permanently blocked |
| Only output | The sole file written is `BRIEF.md` inside the feature folder |
| Re-loop on new info | Any new variable from the user resets to Step 2 |
| Structured format | Always use the three-section format in Steps 2 and 3 |
