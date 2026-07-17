# Examples-for-Confirmation Format (Step 3)

Present the concrete examples using this structure:

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

Reminders for the content of the table:

- **Data minimality** — each row carries only the data that affects the behavior; leave incidental fields out (or mark them `—`).
- **Business language** — describe the situation and outcome as a stakeholder would, not as the system implements it.
