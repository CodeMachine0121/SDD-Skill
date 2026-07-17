---
name: clarify
description: >
  Reach 100% intent consensus with the user, pin the intent down with concrete
  examples that cover boundaries and exceptions, then compile a requirements
  brief written purely in business language for handoff to PRD.
  Never edits source code.
  Triggered by: "clarify", "/clarify".
---

## Core Rule

**Never edit source files.** The only output this skill produces is a requirements brief written to `.sdd/`. The only accepted confirmation phrases are **"Confirm"** or **"Go"**.

**Business language only.** The brief and every example in it describe *what the business needs*, never *how it is built*. No technical terms are allowed anywhere in the output — no code identifiers, class/function/table/endpoint names, HTTP status codes, frameworks, data structures, or any implementation detail. Write in the domain vocabulary a non-engineer stakeholder would use (align with `.sdd/UL-MAP.md`). If a fact can only be stated technically, it is a decision for the PRD, not the brief.

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

### Step 3 — Concrete Examples (Specification by Example)

Once you believe you have captured the developer's intent, do **not** jump straight to abstract requirements. First make the intent concrete: turn the behavior into a short list of **real, specific examples**, then present them for confirmation. A wrong assumption is far easier to catch in a concrete example than in a prose rule.

For each behavior rule, cover at minimum:

- the **happy path** — the ordinary, expected case;
- every **boundary** — the exact edge where behavior changes (the threshold, the first/last valid value, empty/full, min/max);
- every **exception** — inputs or states that trigger a different rule (rejection, error, fallback, special handling).

If a rule has a branch, there must be an example on each side of it. A rule with no accompanying example is not yet agreed.

**Data minimality — the key principle.** Each example carries **only the data that affects the behavior or logic being expressed** — nothing more. Do not list every field of an entity; list the ones that make *this* example produce *this* outcome. Irrelevant data hides the rule and makes the example lie about what matters. If a field does not change the outcome, leave it out (or mark it `—`).

Phrase every example in business language (see the Core Rule) — describe the situation and outcome as a stakeholder would, not as the system implements it.

Present like this:

```
**[Examples for Confirmation]**

Rule: <the behavior rule this set of examples pins down>

| # | Given (only relevant data) | When | Then |
|---|---|---|---|
| 1 (happy)     | <input>  | <action> | <expected outcome> |
| 2 (boundary)  | <input>  | <action> | <expected outcome> |
| 3 (exception) | <input>  | <action> | <expected outcome> |

<repeat one block per rule when there are several>
```

Wait for the user to confirm or correct the examples. Any correction may surface a new rule or hidden branch → return to **Step 2**.

---

### Step 4 — Proposal

After ambiguities are resolved and the examples are confirmed, present the brief outline:

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

Confirmed examples: <N> across <M> rules

Will write to: .sdd/{yyyy-MM-dd}-{feature-slug}/BRIEF.md

Type "Confirm" or "Go" to generate the file.
```

If the user's response introduces new variables or changes scope → return to **Step 2**.

---

### Step 5 — Output

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

## Examples (Specification by Example)
Each example lists **only** the data that affects the behavior — nothing more.

### Rule: <behavior rule>
| # | Given (only relevant data) | When | Then |
|---|---|---|---|
| 1 (happy)     | <input> | <action> | <expected outcome> |
| 2 (boundary)  | <input> | <action> | <expected outcome> |
| 3 (exception) | <input> | <action> | <expected outcome> |

## Out of Scope
- <item>

## Open Decisions
Items the PRD author should resolve:
- <item>

## Context / Background
<any relevant notes from the clarification conversation>
```

4. Before finishing, re-read the brief and confirm it contains **zero technical terms** — every rule and example must read as a business statement. Strip or rephrase anything that leaked implementation detail.
5. Report: "Brief written to `.sdd/{yyyy-MM-dd}-{feature-slug}/BRIEF.md`. Run `/prd` to generate the PRD."

---

## Hard Constraints

| Constraint | Rule |
|---|---|
| No source edits | `Edit`, `Write`, `Bash` on source files are permanently blocked |
| Only output | The sole file written is `BRIEF.md` inside the feature folder |
| Business language only | The brief contains zero technical terms — no code identifiers, APIs, data stores, frameworks, status codes, or implementation detail; only domain/business language |
| Examples before prose | Intent must be pinned with concrete examples (happy path + every boundary + every exception) before the proposal; an un-exampled rule is not agreed |
| Data minimality | Every example carries only the data that affects the behavior being expressed — never a full entity dump |
| Re-loop on new info | Any new variable from the user resets to Step 2 |
| Multiple-choice questions | Every open question offers ≥3 concrete options plus a final "Other — type your own answer" option |
