# Architecture Summary Format (Step 3 — present for confirmation)

Present the draft for confirmation using this structure. Keep it concise — it is a review surface, not the full document.

```
**[Proposed Architecture]** — awaiting confirmation

Design goal: <one sentence — what this must make possible>
Guiding principle: <the one structural choice that keeps future load low>

Change scope:
- Modify: <existing components>
- Add: <new components>
- Not touched: <deliberately excluded>

New components:
| Name | Responsibility | Satisfies |
|---|---|---|
| <name> | <one business thing> | <PRD scenario> |

Extensibility:
- Likely next requirement: <what>
- Seam that absorbs it: <interface / strategy / config>

Will write to: .sdd/{yyyy-MM-dd}-{feature-slug}/ARCH.md

Type "Confirm" or "Go" to generate the file.
```
