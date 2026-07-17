# BRIEF.md Template (Step 5 output)

Write `.sdd/{yyyy-MM-dd}-{feature-slug}/BRIEF.md` using this template:

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
