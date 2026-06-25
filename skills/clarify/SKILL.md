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

### Step 1 — Investigate First

When the user presents a request, **investigate before asking anything**. Do **not** open any editor tool — this step is read-only.

1. **Scan the codebase** for code relevant to the request: search for related modules, entities, functions, and existing patterns. Note how similar things are already done.
2. **Read sibling feature docs.** List `.sdd/` and read related `BRIEF.md` / `PRD.md` files in other feature folders, plus `.sdd/PROJECT.md` and `.sdd/UL-MAP.md` if they exist. Reuse decisions, conventions, and vocabulary already established there instead of re-asking.
3. **Write down** your understanding of the goal in plain language, and an explicit list of what you now know from investigation vs. what is still genuinely unknown.

---

### Step 2 — Clarification (only if needed)

After investigating, decide whether questions are actually necessary:

- If the codebase and existing docs already answer everything and the intent is unambiguous → **skip to Step 3** and state that no clarification was needed.
- Otherwise, ask **only** about what investigation could not resolve. Never ask about something you could have learned from the code or sibling docs.

Identify anything that is still ambiguous, under-specified, or potentially risky. Present using this exact structure:

```
**[Current Understanding]**
<one-paragraph summary of what you think the user wants>

**[From Investigation]**
- <what the codebase / sibling docs already established — patterns, conventions, prior decisions>

**[Open Questions]**  (only what investigation could not resolve)
1. <question>
   A. <option — your recommended choice goes first>
   B. <option>
   C. <option>
   D. Other — type your own answer
2. <question>
   A. <option>
   B. <option>
   C. <option>
   D. Other — type your own answer
…

**[Estimated Scope]**
- Files likely affected: <list>
- Files definitely NOT affected: <list if useful>
```

**Every open question MUST offer at least three concrete options plus one final "Other — type your own answer" option.** Always present at least three distinct, mutually-exclusive choices the user can pick by letter, and always keep the last option open for the user to type a custom answer. Base the options on what you learned during investigation; put the option you recommend first.

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
| Multiple-choice questions | Every open question offers ≥3 concrete options plus a final "Other — type your own answer" option |
