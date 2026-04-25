---
name: ubi
description: >
  Manage a Ubiquitous Language Map — bridges business terms, code names, and UI labels within a Bounded Context.
  Use when: initializing a new UL map, or updating/reconciling an existing one.
  Triggered by: "ubiquitous language", "UL map", "domain glossary", "/ul-init", "/ul-update".
---

## Mode

| Signal | Mode |
|--------|------|
| "init/create/new", no map file found | **INIT** |
| "update/add/sync/reconcile", map file exists | **UPDATE** |

---

## INIT

1. Confirm: project name, bounded context, maintainer, source files to scan.
2. Read source files. Extract candidates for all 4 sections: nouns, actions, ambiguities, enums/magic values.
3. Create `.sdd/` folder at project root if it does not exist.
4. Generate `.sdd/UL-MAP.md` using [references/ul-map-template.md](references/ul-map-template.md). Pre-fill rows from step 2. Mark unverified entries `Archeology`.
5. Report counts: how many `Archeology` vs `Confirmed`. Remind user to verify with domain expert.

---

## UPDATE

1. Read the existing map file at `.sdd/UL-MAP.md` (project root).
2. Apply the requested change — add / refine / resolve / deprecate.
3. Rules:
   - Never delete a row — set status `To Be Deleted` instead.
   - Update `Last Updated` date.
   - When resolving a conflict, update Resolution column and promote status to `Confirmed`.
4. Report diff: rows added / modified / deprecated.

---

## Status Values

| Status | Meaning |
|--------|---------|
| `Archeology` | Found in code; business meaning unverified |
| `Confirmed` | Verified with domain expert |
| `To Be Deleted` | Deprecated; keep row as history |

